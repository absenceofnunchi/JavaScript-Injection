# JavaScript Injection

App extension lets you extend custom functionality and content beyond your app and make it available to users while they’re interacting with other apps ro the system.  Good examples would be a widget of an app or a custom keyboard  in place of the native keyboard.  It’s important to know that an app extension is not an app, according to the Apple documentation.  It implements a specific, well scoped task that adheres to the policies defined by a particular extension point. 

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/4.png" width="400">

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/5.png" width="400">

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/6.png" width="400">


## Features

The app extension explained: 

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/1.png" width="400">

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/2.png" width="400">

<img src="https://github.com/igibliss00/JavaScript-Injection/blob/master/README_assets/3.png" width="400">

([Source](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2))

### Keyboard Safe View

#### NotificationCenter

A notification dispatch mechanism that enables the broadcast of information to registered observers. We’ll be using the addObserver(_:selector:name:object:) method from the instance of the NotificationCenter class to listen to the changes in the keyboard’s state. The addObserver() method will call the adjustForKeyboard() method once the notification has been dispatched. 

```
let notificationCenter = NotificationCenter.default
notificationCenter.addObserver(self, selector: #selector(adjustForKeyboard), name: UIResponder.keyboardWillHideNotification, object: nil)
notificationCenter.addObserver(self, selector: #selector(adjustForKeyboard), name: UIResponder.keyboardWillChangeFrameNotification, object: nil)
```

#### NSValue

Objective-C  doesn’t have a way to store structures like CGRect in an array or a dictionary so Apple created a special case called NSValue that acts as a wrapper around structures so that they could be saved in an array or a dictionary.  When the keyboard has finished animating, a dictionary containing UIResponder.keyboardFrameEndUserInfoKey as the key will return a value of type NSValue, which in turn is of type CGRect that holds a CGPoint and a CGSize of the keyboard. cgRectValue property holds the values of CGRect that we’re ultimately looking to extract.

The notification dictionary -> UIResponder.keyboardFrameEndUserInfoKey as the key of the dictionary -> NSValue as the value of the dictionary ->CGRect -> cgRectValue

In plain english, we asked the notification centre to dispatch two things, keyboardWillHideNotification and keyboardWillChangeFrameNotification, as they change, which will sent out in the form of dictionaries.  We are specifically listening for a particular value whose key is UIResponder.keyboardFrameEndUserInfoKey.  This value is CGRect of the NSValue type.  Finally, CGRect has the cgRectValue that we’re looking for.

```
@objc func adjustForKeyboard(notification: Notification) {
    guard let keyboardValue = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? NSValue else { return }

    let keyboardScreenEndFrame = keyboardValue.cgRectValue
    let keyboardViewEndFrame = view.convert(keyboardScreenEndFrame, from: view.window)

    if notification.name == UIResponder.keyboardWillHideNotification {
        script.contentInset = .zero
    } else {
        script.contentInset = UIEdgeInsets(top: 0, left: 0, bottom: keyboardViewEndFrame.height - view.safeAreaInsets.bottom, right: 0)
    }

    script.scrollIndicatorInsets = script.contentInset

    let selectedRange = script.selectedRange
    script.scrollRangeToVisible(selectedRange)
}
```
