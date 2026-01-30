




## Working with rclone 

Need to pay attention on the rclone.conf is a per user file under user home. 
So usually the hook script is run as root.

https://forum.rclone.org/t/trying-to-make-rclone-conf-available-to-all-users-on-linux/42267

The trick in above link bascially won't work with zrepl because of the protection.

With the `ProtectHome=read-only` can't change anything under home. Also the system hardening would prevent it from changing files in /opt. Even putting it in root home wont work (still protected)

The solution is to add exception in systemd unit, `sudo systemctl edit zrepl` with folloing addition
```
[Service]
ReadWritePaths=/opt/rclone
```





### Post-hook example 
```bash
#!/usr/bin/env bash
set -xeuo pipefail

echo "stdout and stderr re-direct to systemd tag zrepl-hook"

exec > >(systemd-cat -t zrepl-hook -p info)
exec 2> >(systemd-cat -t zrepl-hook -p err)

echo "================================================="
echo "======== SNAPSHOT UPLAOD SCRIPT ================="
echo "==== $(date +\"%Y-%m-%dT%H:%M:%S%:z\") ===="

RCLONE_REMOTE="${RCLONE_REMOTE:-some_rclone_remote}"     # rclone remote (already set up)
REMOTE_ROOT="${REMOTE_ROOT:-PoolData}"       # root folder name on the remote
HOLD_TAG="${HOLD_TAG:-zrepl-rclone-hook}"            # tag for zfs hold/release

echo "hook type: ${ZREPL_HOOKTYPE}"

if [[ "$ZREPL_HOOKTYPE" != "post_snapshot" ]]; then
  echo "Wrong hook type, skip"
  exit 0
fi

# Clean up to prevent hook from too much failed hooks

HOLD_TAKEN=""

cleanup() {
  # Always attempt to release the hold if we took it
  if [[ -n "${HOLD_TAKEN}" ]]; then
    echo "[INFO] Releasing hold for ${ZREPL_FS}@${ZREPL_SNAPNAME}  (tag=${HOLD_TAG})"
    # Don't let release errors hide original exit status
    set +e
    zfs release "${HOLD_TAG}" "${ZREPL_FS}@${ZREPL_SNAPNAME}" || true
  fi
}
trap cleanup EXIT INT TERM


# Hold the snapshot for all the rest of these steps.
zfs hold "${HOLD_TAG}" "${ZREPL_FS}@${ZREPL_SNAPNAME}"
HOLD_TAKEN="1"

SRC_PATH="$(zfs get -H -o value mountpoint ${ZREPL_FS})/.zfs/snapshot/${ZREPL_SNAPNAME}"

ls "${SRC_PATH}"

# Build remote path: replace pool name (first path segment) with $REMOTE_ROOT
case "$ZREPL_FS" in
  */*) REMOTE_PATH="${REMOTE_ROOT}/${ZREPL_FS#*/}" ;;
  *)   REMOTE_PATH="${REMOTE_ROOT}" ;; # This line is for case like `RedPool` without dataset name.
esac
DEST="${RCLONE_REMOTE}:${REMOTE_PATH}"

echo "[INFO] Upload -> ${DEST}"

export RCLONE_CONFIG=/opt/rclone/rclone.conf
rclone sync "${SRC_PATH}/" "${DEST}"

echo "[INFO] Finished uploading"
```