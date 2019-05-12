# NSNotificationCenter-KVO
NSNotificationCenter & KVO
Intro
An iOS program consists of many (hopefully) loosely coupled objects. These objects need to communicate with each other to get stuff done.
If an instance of object A created and so owns an instance of object B, then A can send B any messages that B responds to. This kind of communication is unproblematic. But sometimes we are in situations where we don't want, or don't have, one object owning the object it needs to send messages to. In these cases we will want to pick one of the many communication patterns common in iOS.
There are 5 basic communication patterns: Delegation, Target Action, Blocks, KVO (Key Value Observing) and NSNotificationCenter.
We have covered Delegation and Target Action a bit already. In this exercise we will checkout NSNotificationCenter and KVO.
Do not confuse NSNotificationCenter with user facing notifications that appear on the lock screen and elsewhere. These are completely different things.
NSNotificationCenter
Learning Outcomes
Can understand what NSNotificationCenter is.
Can understand when to use NSNotificationCenter.
Can an add an observer to listen to system notifications and consume userInfo dictionary.
Can post a custom notification with or without a userInfo dictionary.
Can observe a custom notification with or without a userInfo dictionary.
Can use Apple's documentation.
Can use and understand the view controller life cycle.
What is NSNotificationCenter?
NSNotificationCenter is a very simple and powerful mechanism used heavily throughout Cocoa/CocoaTouch to send messages from one object to a relatively unrelated object(s).
NSNotificationCenter is like a radio. One object broadcasts a message (posts them) when some event happens. Other objects can tune in to receive (observe) these messages.
When the receiver gets a message it calls a method using a selector. (You can optionally execute a block. We won't worry about the version of addObserver that takes a block, but feel free to experiment with it on your own).
Custom classes are free to post/observe custom notifications or simply observe system notifications.
NSNotificationCenter implements the Observer Design Pattern.
NSNotificationCenter is created by the system when your app starts. It is an example of the Singleton Design Pattern. At most one instance ever exists. To get ahold of this instance you must call it like this:
NSNotificationCenter *notificationCenter = [NSNotificationCenter defaultCenter];
Note: You never need to have a strong reference to the instance because there is only ever one instance and that instance is retained by the application automatically. So, whenever you need it just use the snippet above to get the reference to it.

Exercise 1: Creating a Custom NSNotification With a userInfo Dictionary
By the end of this exercise your app should look like this: 
Setup Instructions
Create a Tabbar application either progrommatically or using storyboard with two view controllers.
Add a UIStepper controller to the first view controller.
Wire up the stepper to an action method.
You will also need an outlet to get the initial value of the stepper. (You could hard code 0 but I prefer that you get the 0 value from the stepper's value property instead.)
You can create a label at the top of each view controller to identify which one you're in.
The second view controller should have a label in the middle to display the current stepper count.
If you would like to see how I did this, watch this screencast:

Posting an NSNotification
Next please take a look at the documentation for NSNotificationCenter.
The first view controller is going to post a notification whenever the stepper's action fires.
We are going to post the stepper's value using an NSDictionary.
So, in the stepper's action method get a reference to the NSNotificationCenter using the syntax above.
Create an NSNotification instance using the initializer - (instancetype)initWithName:(NSNotificationName)name object:(id)object userInfo:(NSDictionary *)userInfo;
You will have to create an NSDictionary instance to pass to the userInfo parameter of this initializer. Make its key stepperValue (or whatever you like). The value of this key will be the stepper's value property. This is a double. So, convert it to an NSNumber. Remember NSDictionary values must be objects.
The NSNotificationName is just a name you supply that the observer will use to register for this message.
Now post the notification using NSNotificationCenter's method - (void)postNotification:(NSNotification *)notification;. You can see that this is an instance method. It is called on the NSNotificationCenter's instance. You pass it the NSNotification you created.
Observing an NSNotification
Next you will have to add the second view controller as the observer.
To do this you use one of the two methods to add an observer. In this case we will use the method - (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSNotificationName)aName object:(id)anObject;
The observer is going to be the second view controller instance.
You can see that this is an instance method called on the instance of NSNotificationCenter. So, get a reference to it. (Use the code snippet above).
The method also requires a selector. This is the method that will fire when the notification is received. It should take a single parameter. The parameter gives you a reference to the NSNotification object that got posted. So, it is of type NSNotification *.
Inside the selector method use the parameter to access the userInfo dictionary. Grab the value using the key you set. The value with be an NSNumber wrapping a double.
Use this value to set the count label.
It is best practive to remove the view controller as an observer in its dealloc method. dealloc is called immediately prior to the object being killed. Never call super in dealloc when using ARC.
Stretch goal
Get the initial value for the label using the stepper's outlet. To do this call the action method on the stepper yourself in viewDidLoad with the initial value.
This will create a problem you will have to solve. When the second view controller is instantiated it's viewDidLoad method does not fire until you first tap the second tab. You will have to figure out how to add NSNotificationCenter before viewDidLoad fires because you should be able to send notifications to the second view controller before viewDidLoad fires. This means that tracking the count must be done assuming the view has not yet been created. Tracking the count should be independent of being displayed. This means that you must use viewWillAppear to update the view whenever you tap on the tab.
Exercise 2: NSNotificationCenter System Events
Learning Outcomes
Can use system notifications.
Can access userInfo dictionary values.
Can convert from NSValue to CGRect.
Can understand basic view geometry.
Can adjust the constant of an autolayout constraint.
Can dismiss the keyboard using resignFirstResponder.
Requirements and Some Tips
The goal is to build a view controller that uses a system notification to get the height of the keyboard. We will use this value to move a view out of the way of the keyboard. This is not necessarily the way you will handle keyboard overlay problems, but it will give you the pieces you need to solve this in any app you build.
Here's a little screencast that I use to expain some of the requirements and how this might look.

Some Notes
When you add yourself as an observer for the keyboard notification you will want to dump these values to the console. The observer takes the key UIKeyboardWillChangeFrameNotification. The name makes sense because we want to change the bottom constraint depending on the frame the keyboard will have once it moves. If we waited until the keyboard already moved our view would appear to come from underneath the keyboard, which is probably not what we want.
When we add an observer the parameter is an NSNotification that contains all of the data we need. What we're looking for is the frame the keyboard will have. When we tap the textField we want the frame the keyboard will have when it finishes moving up. We can compare the keyboard's y value to the height of self.view. If they are the same then we know the keyboard is hidden or about to be hidden.
Notice that the userInfo dictionary on the keyboard notification returns an NSValue that's wrapping the CGRect. You add a CGRect (which is a C Struct) to an NSDictionary value by wrapping it in NSValue, since the values of NSDictionaries must be objects not C types. So, you need to unwrap this NSValue to get at the CGRect value.
Finally, remember to remove yourself as an observer in dealloc. Look up the syntax in NSNotificationCenter's documentation.
Tip: If your keyboard doesn't come up in the simulator, check the Simulator's Hardware > Keyboard menu and toggle the keyboard on there.

References for NSNotificationCenter
NSNotificationCenter Class Reference

NSHipster Post

KVO
Learning Outcomes
Can use KVO as a communication mechanism.
Can userstand when to use KVO.
Can implement a custom UIView subclass and allow the subclass to manage its own gestures.
Can use UIPanGestureRecognizer to move a view using the translation attribute.
Can convert from NSValue to CGPoint.
Can change a CGPoint value in Objective-C.
KVO Basic Concepts
Key value observing is a simple mechanism that allows an object to watch for changes to state on one or more properties of another object.
It is used when one class A has a reference to another class B, and where A would like to know about changes to B but where B has no knowledge another object is even watching its properties. It therefore knows nothing about the identity of the observer.
KVO uses an observer pattern that is common to many modern event driven frameworks especially popular in javascipt frameworks.
KVO, like notification center, is a great way to handle asynchronous calls.
All subclasses of NSObject have this capacity built in.
KVO is used less in iOS than Mac development, but it is still used and you will be asked about it in interviews. (BTW, KVO is not really a good option in Swift at least for now, but this could change.)
Observing State Changes Using KVO
To add yourself as an observer you need to call observeValueForKeyPath:ofObject:change:context:. The keyPath is a string representation of the property. In our case, the property is an CGPoint called point. So the keyPath is simply @"point".
Like NSNotificationCenter you should remove yourself as an observer in dealloc by calling removeObserver:forKeyPath:context:.
Unlike NSNotificationCenter you do not specify a selector and a custom method that is called when a message is received. All messages come through a single method: observeValueForKeyPath:ofObject:change:context:. So, you must implement this method on observer (in our case, this is FourthViewController).
Exercise 3 Outcome and Goal
Please view the following screencast to get the requirements for the KVO exercise, to get some tips and to see the final result in action.

Note: You should read this NSHipster article on KVO first before doing the exercise.

Final Goal and Reading
objc.io has a very good article on KVO here.
You should be starting to get a better sense of the main communication patterns you will be using as an iOS developer. You should start to consider when and why you should use one pattern over another. This is a very common interview question. I highly recommend reading this important article on the topic.
