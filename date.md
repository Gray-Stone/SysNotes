# Date command!

Date is a useful command to handle time and date processing.

Just type `Date` to get the current time
```
$ date
Fri 13 May 2022 04:52:31 PM EDT
```

To get time stamp in ISO format 
```
$ date -Is
2022-05-13T16:51:33-04:00
```

Date in epoch time:
```
$  date +%s
1652475183
```

## Any format

Date can output any format by using the `+` syntax.

## Convert between formats

It a common thing to trying to convert between time format.

Example to convert ISO format to epoch
```
$ date -d "2022-05-13T13:00:10-0400" +%s
1652461210
```

And to convert back
```
$ date -d "@1652461210" -Is
2022-05-13T13:00:10-04:00
```


