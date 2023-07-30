This file will be moved to other repo later. 


## C callback with cpp

In C library's API, the callback function pointer in these API will only take free function or static member. 

For non static member, there is the impliciet information of the object instance. Which the C API function pointer will not carry. Thus not possible to use a member function in this case.

https://stackoverflow.com/a/53643286

With a well-defined C library. It's API will accepct a user data pointer along side the call-back, usually with type *void. This user data will also be passed into the call-back. Which mean one can recover the object instance information from this user data, then call the member function using it.