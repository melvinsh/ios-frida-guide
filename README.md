# ios-frida-guide

## Objection
Finding process:
```
➜  iOS frida-ps -U
PID  Name
---  ----------------
211  Mail
 74  BTServer
359  CloudKeychainPro
```

Exploring with Objection.
```
➜  iOS objection --gadget 'Mail' explore

     _     _         _   _
 ___| |_  |_|___ ___| |_|_|___ ___
| . | . | | | -_|  _|  _| | . |   |
|___|___|_| |___|___|_| |_|___|_|_|
        |___|(object)inject(ion) v1.2.2

     Runtime Mobile Exploration
        by: @leonjza from @sensepost

[tab] for command suggestions
com.apple.mobilemail on (iPad: 9.3.5) [usb] # 
```

Finding classes.
```
com.apple.mobilemail on (iPad: 9.3.5) [usb] # ios hooking search classes Password
UIDocumentPasswordView
UIDocumentPasswordField
WBSSavedPasswordStore
WBSSavedPassword
WBSPasswordGeneration
```

Watching classes.
```
com.apple.mobilemail on (iPad: 9.3.5) [usb] # ios hooking watch class UIDocumentPasswordView
Job: fffffb3e-2ed1-43bb-a384-4afc7eb00e23 - Starting
[4afc7eb00e23] [call-to-hooked-method] Found class: UIDocumentPasswordView, hooking methods...
Job: fffffb3e-2ed1-43bb-a384-4afc7eb00e23 - Started
```

Finding class methods.
```
com.apple.mobilemail on (iPad: 9.3.5) [usb] # ios hooking list class_methods UIDocumentPasswordView
- drawRect:
- dealloc
- layoutSubviews
- _canDrawContent
- textFieldDidBeginEditing:
- textFieldDidEndEditing:
```

Changing return value.
```
com.apple.mobilemail on (iPad: 9.3.5) [usb] # ios hooking set return_value "*[UIDocumentPasswordView passwordChangeRequire:]" true      
```

## Frida scripts
Logging arguments.
``` javascript
var checkPassword = ObjC.classes.PasswordManager["- setNewPassword:withOldPassword:"];

Interceptor.attach(checkHistory.implementation, {
  onEnter: function(args) {
    // args[0] is self
    // args[1] is selector
    // args[2] holds the first function argument, an NSString
    var new_password = ObjC.Object(args[2]);
    console.log(new_password);

    var old_password = ObjC.Object(args[3]);
    console.log(old_password);
  }
});
```

Overwriting return value.
``` javascript
var checkPin = ObjC.classes.PinScreen["- pinMatched:store:"];

Interceptor.attach(checkPin.implementation, {
  onLeave: function(retval) {
    retval.replace(0x0);
  }
});
```
