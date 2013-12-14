# Objective-C

`%@` in a string means that it calls an implemented method `description`

Arrays only store objects
- Inserting a hole means putting in a `[NSNull null]` not `nil`

Every class is a subclass of `NSObject`
- Implements basic `alloc`, `init`, and `description`

For example `NSMutableArray` extends `NSArray` and holds pointers to objects dynamically

Header files (`.h`) also called *interface files* and implementation files (`.m`)

Declaring a class means `@interface <ClassName> : <SuperClass>`

Setters are `setItemName` and getters are just the property `itemName`

`isa` is a pointer to the class the object identifes with

`id` is an opaque type (often used with init) so that subclasses can override (cant have same method name with selector and different return types)

`self` is an implicit local variable (similar to `this`)

Check that `super` calls are not nil

`init` rules

- All classes inherit from it's superclass `[super init]`
- All classes pick a designated initializer (takes all arguments)
- Designated initializer calls `super`
- Other initializers only call designated initializer

`super` is the reference to the superclass!

# Managing memory with ARC

ARC: automatic reference counting

## Heap

`alloc` throws the object onto the heap

NSDate has 12 bytes (double for `elapsed` time and isa for class storage)

Objects exist separately on the heap (`BNRItem` has a reference to the `NSDate`)

Objects keep references and store the *address* of the object (a pointer)


## Stack

Stores frames in memory

Function/method invocation takes a frame (chunk of memory)

Stores values for variables

Frames stack on top of each other as a method can call another method etc.

Stack is popped off after it's returned

## Pointer variables and object ownership

- Method has local variable that points to object, method *owns* object
- Object has instance variable to another object, object *owns* that object

## Memory management

Heap is limited, especially on iOS devices, need to **destroy** the objects that aren't needed

To determine whether to destroy an object or not...

- Does it have any owners? If not, it should be destroyed (this is a *memory leak*)
- If it does have an owner, do not destroy it! (if this happens it's known as *premature deallocation*)

To have iOS do this for us, we enable ARC (used to be manual reference counting with `retain` and `release`)

### Losing owners

- Variable points to a different object
- Variable set to nil
- Variable itself is destroyed
- Frame popped loses all variables in that stack context

## Strong and weak references

Pointer to an object claims ownership - this is a *strong reference*

Can have *weak references* where ownership is not claimed

- useful in *retain cycle* (objects strongly refer to each other, ARC will never catch this)

Adding attribute `__weak` will ensure that retain cycles are broken

Think of this as a parent-child relationship, the child should have a weak reference to the parent and the parent know who it's children are

`__unsafe_unretained` means that it will not let the parent be set to `nil`

## Properties

Declaring a `@property` in your code sets up the `setX` and `X` accessor methods

The properties can also have attributes:

```obj-c
@property (nonatomic, readwrite, strong) NSString *itemName;
```

`readwrite` by default, `read` only initializes the getter

Last option determines memory management (primitives is `assign`, objects defaults to `strong`)

Instead of `strong` use `copy` for mutable fields that others may touch

Implicitly sets the setter method to `copy` the field:

```obj-c
- (void)setItemName:(NSString *)str
{
  itemName = [str copy];
}
```

### Dot syntax

An alternative to sending accessor messages to objects is the traditional dot syntax...

```obj-c
[item setValueInDollars:3];
// same as...
item.setValueInDollars = 3;
```

## Autorelease pool and ARC history

Before arc was manual reference counting, would `release` when an owner is lost and `retain` when an owner is gained

Forgetting to `release` meant a memory leak!

People used Clang static analyzer in Xcode to determine whether there were any premature deallocations or memory leaks

Apple integrated Clang as ARC to do the retain and release messages for them

`autorelease` was a mechanism that returned the ownership from the method, if you added it as a message the caller is responsible for the object

The `@autoreleasepool` directive said that any method that didn't have `alloc` or `copy` will be `release`ed at the end of the block

# Delegation and Core Location

In the 'Build Phases' part of Xcode we can see how the project is assembled, added CoreLocation here

Assigning a delegate means that the class will call an instance variable `delegate` with associated methods

- For example, `CLLocationManager` calls `locationManager:didUpdateToLocation:fromLocation` with the `delegate`

## Delegation

Basically object-oriented approach to callbacks, assign an opaque type to `delegate` and that delegate *should* implement the methods you want (they're just callbacks)

Different than target-action pairs with UIButtons and other UI elements

- Assigns a target object that sends action message when an event occurs

Messages from target-actions can be anything, for delegation the object must conform to a set of methods in a *protocol*

When implementing methods from the protocol, the class is said to *conform* to the protocol

```obj-c
@protocol CLLocationManagerDelegate <NSObject>

@optional

- (void)locationManager:(CLLocationManager *)manager
    didUpdateToLocation:(CLLocation *)newLocation
           fromLocation:(CLLocation *)oldLocation;

@end
```

### Protocol methods

There are two kinds of protocol methods:

- Handle information updates (`locationManager:didEnterRegion:`)
- Handle requests for input (`locationManagerShouldDisplayHeadingCalibration`)

We can check if an object responds to a method by using `respondsToSelector`, all objects implement this

Use the `@selector` directive with `respondToSelector`!

```obj-c
- (void)finishedFindingLocation:(CLLocation *) newLocation
{
  SEL updateMethod = @selector(locationManager:didUpdateToLocation:fromLocation:);

  if ([[self delegate] respondsToSelector:updateMethod]) {
    // do some stuff
  }
}
```

Declare that you conform to a protocol in the `@interface` directive of the header file

```obj-c
@interface WhatwhatController: UIViewController <CLLocationManagerDelegate>
```

### Delegation, controllers, and memory management

Controllers own objects, objects delegate to controllers

To avoid retain cycles the `delegate` properties are not strong references (actually `__unsafe_unretained` for legacy non-ARC versions of iOS)

Because of this, we actually have to dealloc the delegates in the controllers!

## Build phases, compiler errors, linker errors

Build phases is what happens when an application is built:

- **Compiler sources**: compiles source code files that are added to the project
- **Link binary with library**: links binaries to libraries so that frameworks can be used
- **Copy bundle resources**: resultant executable is placed in an *application bundle* (folder), resources are listed alongside and may be additional XIBs/images/etc. for the application to use

### Preprocessing

Broken into two steps for compilation: preprocessing and compiling

Preprocessing creations an intermediate file for each `.m` file. It's Objective-C still, just huge.

Takes care of all preprocessor directives (gets an `#import` and throws in the header file)

### Compiling

The generated intermediate file is compiled into machine code and goes into an object file. Errors caught here are known as *compilation errors* and typically involve syntax and static analysis checks.

### Linking

Your compiled file contains the machine code for what *you implemented*, but still needs other implementation files for the frameworks

The compiler leaves a link (Link Binary With Library phase) to this generated code for the library

Basically takes the pre-generated code, mashes the implementation file the user has created, and the resultant binary is our application!

# Subclassing UIView and UIScrollView

A view is...

- Instance of UIView or subclass
- Knows how to draw itself on UIWindow
- Exists within hierarchy of views, root being window
- Handles events (touches)

Subclass of UIView is UIWindow, one instance manages the container for all the views in the application

- All views are subview of the window
- Subviews can have other subviews

Doing UI boils down to creating the image for each view and assembling the hierarchy, UIButton/MKMapView/etc know their layout and image, when they don't we must...

## Create a custom view

Adding the subview in `didFinishLaunchingWithOptions` by using `[[self window] addSubView: view]`

The subview will know who the `superview` is and it is declared as `__unsafe_unretained`

`initWithFrame` takes a `CGRect` that defines the *frame* of the view

- **frame**: size and position of the view *relative* to the superview (members `origin` and `size`)

Secret sauce to having a custom view is `drawRect:`

Need to have a drawing context in the view (maintains state of drawing, color, thickness of pen) and performs the operations

## CoreGraphics

Framework in C that provides an API for 2D drawing

Hub is `CGContextRef:`, all functions/types interact with the context and the context renders the image

Set up state with `SetLineWidth`, color with `SetRGBStrokeColor` and the path to the context by using `CGContextAddArc`

After that we can push the operation:

- Draw a line (StrokePath)
- Draw a shape (FillPath)
- Limit future operations by path (ContextClip)

## Redrawing view

When `UIView` receives `setNeedsDisplay` it will redraw the image

- Views do it for context changes
- UILabel for example does it when `setText:` is called

Redraws are on the run loop, the handler methods are dispatched as the loop goes around and return back to the loop

iOS redraws only images it needs, composits and reuses the previous images

## Motion Events

`UIResponder` is first responder of the window and receives events from shaking the phone, keyboard, etc.

Send the responder `becomeFirstResponder`

## Using UIScrollView

Can be larger than the viewport of the window and allows the user to pan up and down the view

### Panning and paging

The user can pan between two views by making the width the same and adding another subview

UIScrollView has a method to snap views called `setPagingEnabled:` that will snap the view sections

# View Controllers

1 controller : 1 screen

## UIViewController

Specializes in a *single screen*

- `view` property points to an instance of the screen
- Fullscreen view or has subviews

View Controller *creates* the view hierarchy

`UIScreen:mainScreen:` to get the full screen of what's in view as a `CGRect`

`setRootViewController` adds the view as the subview for the window

## Another UIViewController

### .xib files

Adding elements to the UI generates an xib object that is read by the application, easier to do subviews and nested view hierarchies like this

File's Owner: view controller loads view, sets `view` property and places it on the screen

- Can do this with `loadView` and set it at compile-time
- Instantiating ViewController with a view that has a XIB will create the XIB but doesn't know objects is in it's view
- Files Owner becomes a hole in the XOB and is linked when the object is created

### View controllers/loading views

View controller controls view
Each view controller's `view` has subviews, which compoase a hierarchy

Can get its view in two ways:

- View controller with no subviews will override `loadView` to create a new instance and use `setView:` to itself
- Complex hierarchies load view from XIB file

## UITabBarController

Maintains an array of `UIViewController`s and shows a tab for each one

Has a view property that is a blank UIView with a tab bar and view of the selected controller

## View Controller Liefcycle

Begins when it is `alloc` and `init`ed. Will see view generated and destroyed

### Initializing view components

Basically passing the bundle name and the nib name is `nil` since iOS will look for the main application bundle and the nib name that is the name of the current ViewController

### UIViewController and lazy loading

The controller won't create the view until it's brought onto the screen. That means we can't do this during init!

`viewDidLoad` is the method that is called when the view is actually loaded

- Won't be called again unless the view is destroyed when memory is low and the view isn't being shown

`loadView:` defaults to using the XIB file specified in `initWithNibName:bundle:`

ANY extra setup that is related to the view should go into `viewDidLoad:` as it will happen every single time the view is shown on the screen, setting up UI in `initWithNibName:bundle:` will break when memory purges the view object out of memory and maintains the ViewController instance

### Appearing and disappearing views

- **viewWillAppear**: about to be added
- **viewDidAppear**: view has been added
- **viewWillDisappear**: view is about to disappear
- **viewDidDisappear**: view has disappeared

Useful since the ViewController is created once the view is shown/hidden multiple times

## main function and UIApplication

The `main.m` file for any application kicks off the app by creating an instance of `UIApplication`. There is only one instance that maintains the run loop.

It also assigns the Delegate for the UIApplication (in this case the `HypnoAppDelegate` from creating the project). Once it is done loading it kicks off `application:didFinishLaunchingWithOptions:`.

## Retina displays

Using vector graphs such as `drawRect:` doesn't matter when the pixel density is bumped up.

However CoreGrahics uses Quartz which works in terms of points. On retina displays it is 2x2 pixels, non-retina is 1x1.

Bitmapped images (JPEGs and PNGs) don't look so nice, it will require a large file to look "smooth" on the retina display.

(Adding @2x will use this message instead when loading in with `[UIImage imageNamed:]`)

# Notification and rotation

## Notification Center

Every app has an instance of `NSNotificationCenter`

Basically a pub/sub, objects can subscribe and the notification center will publish to subscribers

Published notifications are `NSNotification`s, each one has a name and pointer to object that posted it

- Subscribers can listen for notification names, posting object, or a method you want formatted when a notification you care about is published

```obj-c
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
[nc addObserver:self selector:@selector(retrieveDog:) name:@"LostDog" object:nil];
```

The selector added above will be the callback for the observer.

`nil` is a wildcard

**The notification does not keep strong references**. If an object is posted to that doesn't exist, the application will crash. The Object will have to remove itself as an observe in the `dealloc` method.

```obj-c
- (void)dealloc
{
  [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

## UIDevice notifications

The UIDevice will post notifications regularly.

Here are a few:
- UIDeviceOrientationDidChangeNotification
- UIDeviceBatteryStateDidChangeNotification

## Autorotation

Simply overriding `shouldAutorotate` will enable autorotation

Handling all the subviews when a view is rotated depends on the *autoresizig mask*. This iw how a specific subview will be resized.

Enabling *autoresizing* will allow the struts and springs to be adjusting. These will affect the *autoresizing mask*.


# UITableView and UITableViewController

`UITableView` displays a single column of data with variable rows (think Settings application)

## UITableViewController

- `UITableView` needs a view controller to handle the appearance
- Requires a *data source* for number of rows/data in rows/etc. (Simply an object that conforms to the `UITableViewDataSource` protocol)
- Requires a *delegate* to handle the events that happen on the view (again, conforms to the `UITableViewDelegate` protocol)

UITableViewController has a `view` property that is *always* an instance of `UITableView` (`dataSource` and `delegate` automatically point to the UITableViewController)

## UITableView's data source

`BNRItemStore` will abstract the mutable array of items we have, we put it behind a *static variable* that will only be declared once and never destroyed.

Singletons should override `allocWithZone:` to prevent a user from creating their own instance of the class outside of the singleton method

Needs to call the `super` of NSObject so that there is no infinite loop between `allocWithZone:` and `sharedStore:`

`@class` directive at the top of the file means that compile time is drastically reduced since it doesn't need to pull in the actual header files

Only import the headers when you need to actually use the class

### Hooking up a data source

Hooking up to a data source means that we conform to the `UITableViewDataSource` protocol and implement the required methods.

There are two requirements that the protocol is looking for:

- The number of rows in a section (`tableView:numberOfRowsInSection`)
- The data for an item at a row index (`tableView:cellForRowAtIndexPath`)

## UITableViewCell

The UITableViewCell is the subclass of UIView for the TableView

Has one view: `contentView`

Has three subviews in the `contentView`:

- Two labels (textLabel and detailTextLabel)
- One image (imageView)

Cells can be reused when they are moved on and off the screen

The number of cells on screen are pooled by the reuse identifier that is passed into `cellForRowAtIndexPath`

Idiomatic way to create a table cell..

```obj-c
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"UITableViewCell"];

if (!cell) {
    cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"UITableViewCell"];
}
```

# Editing UITableView

## Editing mode

UITableView has an `editing` property - set it to YES and enter editing mode!

Link IBOutlet to connect *to* a specific component
Link methods that return IBActions *from* a compoennt
Top level objects in the `.xib` file must be a strong reference

Previously `.xib` was loaded automatically by the implementation class existing.

If it needs to be loaded manually: use `NSBundle` (interface between application and application bundle it lives in)

```obj-c
if (!headerView) {
    [[NSBundle mainBundle] loadNibNamed:@"HeaderView" owner:self options:nil];
}

return headerView;
```

The controller gets the message for `toggleEditMode` where the `sender` is the view, it toggles the `editing` property on the view when tapped

## Adding rows

The green plus icon method used to be used, it has fallen out of favor in place of 'New' buttons

When an item is added we need to update the `dataSource` with the number of items that are in the table

## Deleting rows

The `dataSource` also must be updated here when the action is performed in the user interface

`removeObject` looks for equality in `isEqual:`

`removeObjectIdenticalTo:` looks for the exact item to remove before doing so

The `UITableViewDataSource` protocol has a method `tableView:commitEditingStyle:forRowAtIndexPath`

## Moving rows

Need to implement the data storage layer switch if needed

The tableView callback is `tableView:moveRowAtIndex:toIndexPath:`

# UINavigationController

Defined flows (multiple related screens, this is a *drill-down interface*)

The multiple screens are maintained by the UINavigationController in a stack of UIViewControllers (head of stack is visible)

Initialized with one instance of UIViewController - this is the *root view controller*

- The `UINavigiationController` has a property `viewControllers` (NSArray)
- Items are added to the end of the array - which is the visible view (the `topViewController` property)

Cocoa touch does all the heavy lifting all we need to do is initialize the controller and set the root view controller and put that on the window

```obj-c
UINavigationController *nc = [[UINavigationController alloc] initWithRootViewController:ivc];
[[self window] setRootViewController:nc];
```

Bad `.xib` files cause the run time exceptions when the view looks for a variable with that name

Control-drag from interface to header file to declare `IBOutlet`
Control-drag from interface to File Owner to set up `delegate` property

The other view controllers need to be added dynamically since all possible views cannot be created at the start of the application

In our case the `ItemsViewController` knows when a row is tapped and has a reference to the `navigationController` to push

The root controller that is handling the view change should also have access to the underlying model to pass to that next view

`endEditing:` is used to resign the first responder and puts the keyboard away

`UINavigationItem` is the part at the top of the navigation controller (part of the ViewController)

It is composed of 4 items:

- left button
- title view
- right button

# Camera

`UIImageView` to display an image

`UIToolbar` for tool bar at the bottom that can add `UIBarButtonItem`s

Taking a picture uses the `UIImagePickerController` that assigns `sourceType` and delegates

sourceType tells picker where to pull images from

- `SourceTypeCamera`: from the camera
- `SourceTypePhotoLibrary`: from an album
- `SourceTypeSavedPhotosAlbum`: most recently taken photos

Needs to assign a delegate for the callback `imagePickerController:didFinishPickingMediaWithInfo:`

`UIImagePickerController` is a modal view, will pop up and go down

```obj-c
- (IBAction)takePicture:(id)sender {
    UIImagePickerController *imagePicker = [[UIImagePickerController alloc] init];

    if ([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {
        [imagePicker setSourceType:UIImagePickerControllerSourceTypeCamera];
    } else {
        [imagePicker setSourceType:UIImagePickerControllerSourceTypePhotoLibrary];
    }

    [imagePicker setDelegate:self];

    [self presentViewController:imagePicker animated:YES completion:nil];
}
```

The image may or may not be present in the callback (`didFinishPickingMediaWithInfo:`), to mitigate the risk of having a low memory error it is a best practice to put the images in a store for the ViewController to use later, it should always be such that the subviews repopulate when sent the selector `viewWillAppear:`.

Singleton:

```obj-c
+ (id)allocWithZone:(struct _NSZone *)zone
{
    return [self sharedStored];
}

+ (BNRImageStore *)sharedStore
{
    static BNRImageStore *sharedStore = nil;

    if (!sharedStore) {
        sharedStore = [[super allocWithZone:NULL] init];
    }
    return sharedStore;
}

- (id)init
{
    self = [super init];
    if (self) {
        dictionary = [[NSMutableDictionary alloc] init];
    }
    return self;
}
```

## NSDictionary

`NSMutableDictionary` is basically same as `NSDictionary` except has dynamic adding of key/value pairs

Going to defer to Coacoa Touch for creating UUIDs for each of the images that will be the key indexed on the dictionary

```obj-c
CFUUIDRef newUniqueId = CFUUIDCreate(kCFAllocatorDefault);
CFStringRef newUniqueIdString = CFUUIDCreateString(kCFAllocatorDefault, newUniqueId);
NSString *key = (__bridge NSString *)newUniqueIdString;
```

CFUUIDRef comes from Core Foundation and `Ref` is simply a pointer

Creating the object involves specifying the amount of allocated space, usually default is fine (kCFAllocatorDefault)

Using the new unique ID it is cast into a string using `CFUUIDCreateString` with the previously defined ID

Now that we have a CFStringRef, it needs to turn into a NSString. Many Core Foundation classes are *toll-free bridged* with Objective-C counter-parts (CFStringRef -> NSString)

### Core Foundation and toll-free bridging

ARC knows about Objective-C (objects losing an owner, being destroyed, etc.), not the case with Core Foundation.

When a Core Foundation object loses a pointer, you must release it before to prevent a memory leak.

Core foundation memory management:

- variable owns object if function has `Create` or `Copy` in it
- if pointer has a CF object, it *must* be released before that pointer is gone
- after it is released, the pointer is **dead**

ARC freaks out when it sees a cast from a CF to Obj-C object, must use `__bridge` to tell it not to worry about reference counting

## Keyboard events

Converting the UIView to a UIControl will give the user control over tap events, use these to `endEditing` and in the case of `textFieldShouldReturn` from the `UITextFieldDelegate` it should `resignFirstResponder`.

# UIPopoverController and Modal View Controllers

## Universalizing

Sometimes making stuff for iPad is annoying, needs a separate `.xib` file, use `~ipad.xib` convention.

### Determining device family

Need to check by asking `UIDevice:currentDevice:`

Only two values `UIUserInterfaceIdiomPad` and `UIUserInterfaceIdiomPhone`

## UIPopoverController

On the iPhone the screens pop in and out (even the UIImagePickerController modally), on the iPad we are graciously given the opportunity to seize UIPopoverController.

Basically floats over the interface, we set the popover controller's `contentViewController`

Pattern: hitting new* should assign the new `DetailViewController` and have the `UINavigationController` allocated instance set the root controller to the `DetailViewController`

```obj-c
BNRItem *newItem = [[BNRItemStore sharedStore] createItem];

DetailViewController *dvc = [[DetailViewController alloc] initForNewItem:YES];
[dvc setItem:newItem];

UINavigationController *unc = [[UINavigationController alloc] initWithRootViewController:dvc];
[self presentViewController:unc animated:YES completion:nil];
```

Pattern: hooking up the `UIBarButtonItem` to a selector should mean that the selected method will choose to do as they please, in this case dismiss the ViewController

```obj-c
UIBarButtonItem *cancelItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemCancel target:self action:@selector(cancel:)];
[[self navigationItem] setLeftBarButtonItem:cancelItem];

// Other calling code...
[[self presentingViewController] dismissViewControllerAnimated:YES completion:nil];
```

Methods in @selector declaratives and other types of run-time execution contexts won't need to worry about declaration in the header file so long as the method calls are not explicitly being sent as messages to the item.

## Modal view controller styles

Default behavior on small devices takes up the whole screen

iPad gets the option to take up a form sheet or page sheet style.

- Form sheet is the small popup that dims the background
- Page sheet is the default full screen (in portrait, in landscape it will dim left and right edges to maintain dimensions)

FormSheet means that the view hasn't disappeared (viewWillAppear/didAppear hooks aren't called, which we used to reload the table)

The place to insert it here is `dismissedViewControllerAnimated:completion:`

## Completion blocks

Basically a closure that has the context of where it's called

```obj-c
// As a property
@property (nonatomic, copy) void (^dismissBlock)(void);

// As an argument
[dvc setDismissBlock:^{
    [[self tableView] reloadData];
}];
```

## Modal view controller transitions

Can set the `modalTransitionStyle` for how the view comes into the screen

## View Controller Relationships

Two types: *parent-child* and *presenting-presenter*

### Parent-child relationship

Use of view controller containers (UINavigationController, UITabBarController, UISplitViewController)

- Can tell by `viewControllers` array that has more view controllers
- Subclasses `UIViewController`

These view controller relationships compose a family, where a container may contain another container

### Presenting-presenter relationship

Occurs when a view is presented modally (view controller is said to be *on top of* view that presents it)

Modally presented view controllers have a property `presentingViewController` that points to the presenting controller

Presenters have a pointer to what is on top of it through the `presentedViewController` property

### Inter-family relationships

The communication of composition between families occurs with the oldest ancestor carrying out the task. This was a reason that `DetailViewController` took over for actions that was happening when the `ItemsViewController` was being touched (and thus that was responsible for setting the block that's executed during the presentee going away)

# Saving, Loading and Application States

## Archiving

Most of the time we are talking about saving and loading model objects, where the data comes from in the interface

Use of *archiving* to save and load BNRItems between usages of the application

- Most common way on iOS
- Records all instance variables and puts them on the file system
- Unmarshals the data off disk and hydrates the objects in memory

Archival conforms to the `NSCoding` protocol and must implement `encodeWithCoder` and `initWithCoder`

`encodeWithCoder` will need to put all of the object properties into the coder that is passed ino

```obj-c
- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:itemName forKey:@"itemName"];
}

// Decoder will hydrate the object again

- (id)initWithCoder:(NSCoder *)aDecoder
{
    self = [super init];
    if (self) {
        [self setItemName:[aDecoder decodeObjectForKey:@"itemName"]];
    }
    return self;
}
```

`initWithCoder` does not play into *use the specific init* functions! Leave it alone, simply rehydrates model objects.

Similar hydration process occurs when `.xib` files are generated in the UI. Marshaled and written to disk.

## Application sandbox

The application sandbox is a barricaded folder for the application on the file system. You play in it, stay there and nobody else will come in.

Directory name      | File description
--------------------| -----------------------------------------
app bundle          | contains resources/executable. Read only
Library/Preferences | Preferences are stored here, `NSUserDefaults` is here, synchronized with iTunes/iCloud
tmp/                | Scratch directory during run time, OS will purge when app is not running. `NSTemporaryDirectory` provides path to this folder.
Documents/          | Write data generated at runtime, backed up and synchronized with iTunes/iCloud. Has restoration mechanism.
Library/Caches      | A place to persist during runtime, not synchronized. Useful for data that is stored on a server and can be retrieved there if the cache is flushed.

### Constructing file path

Use `NSSearchPathForDirectoriesInDomain` to look around the sandbox, it returns an array of strings (only one in iOS) for the user to look up

## NSKeyedArchiver and NSKeyedUnarchiver

When the application *exits* and *enters* it's a good time to save stuff. Use `NSKeyedArchiver` during exit, `NSKeyedUnarchiver` during enter

```obj-c
// It's this simple...
NSString *path = [self itemArchivePath];
return [NSKeyedArchiver archiveRootObject:allItems toFile:path];
```

It basically...

1. Creates an instance of `NSKeyedArchiver`
2. Sends `encodeWithCoder` to the root object (`allItems`)
3. Each item in the `allItems` calls `encodeWithCoder` and passes the same `NSKeyedArchiver`
4. Gets the output object and writes it to the path

Hook to use here is `applicationDidEnterBackground`

Rehydrating the items simply means that when the `BNRItemStore` is initialized we take the items out at the path...

```obj-c
NSString *path = [self itemArchivePath];
allItems = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
```

Handy thing is that the archivers handle identification of objects for you, no need to worry about that.

## Writing to filesystem with NSData

Writing the image to disk is what we want to do since it's only in the application.

Persisted with NSData class which encapsulates creating/maintaining/releasing a manual buffer

```obj-c
NSString *imagePath = [self imagePathForKey:s];
NSData *d = UIImageJPEGRepresentation(i, 0.5);

// This is a binary write to the filesystem not an archival
[d writeToFile:imagePath atomically:YES];
```

Wipe the file from disk when the model is deleted as well...

```obj-c
NSString *imagePath = [self imagePathForKey:s];
[[NSFileManager defaultManager] removeItemAtPath:imagePath error:NULL];
```

### More on low memory warnings

Message comes in `didReceiveMemoryWarning` and destroys views that are not in screen. Purges what's not immediately being used. Hook into this through the `NSNotificationCenter` and clear the cache of objects that we don't need.

```obj-c
NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
[nc addObserver:self selector:@selector(clearCache:) name:UIApplicationDidReceiveMemoryWarningNotification object:nil];
```

## Model-View-Controller-Store pattern

Traditional MVC the controller has to worry about the model persistence and whether it will get actual items from querying, etc.

The abstraction of a store allows for the controller to fetch and save objects without having to be tangled in what is actually going on (archival, writing to filesyste, syncing with iCloud, etc)

This is powerful because we can switch out the underlying implementation details of what is going on in the code without affecting controller code.

This is known as MVCS (model-view-controller-store)

## Reading and writing from the file system

A couple other ways to read and write from the file system in addition to `NSData`. CoreData, and some others...

Standard C is allowed here..

```c
FILE *inFile = fopen("textfile", "rt");
char *buffer = malloc(somesize);
fread(buffer, byteCount, 1, inFile);
```

NSData is a good replacement for this. Works for binary data well.

`NSString` has `writeToFile:atomically:encodingerror:` and `initWithContentsOfFile:`

`NSDictionary` and `NSArray` have `writeToFile` and `initWithContentsOfFile` to persist to the file system. Must contain only *property list serializable* objects such as NSString, NSNumber, NSDate, NSData, NSArray, NSDictionary


# Subclassing UITableViewCell

Most of the time `textLabel`, `detailTextLabel`, and `imageView` is okay, sometimes we need to subclass it however.

Subclassing `UIView` means overriding the `drawRect:` method to edit the views appearance

`UITableViewCell` is not like this, each "cell" has a `contentView`

Need to use `contentView` to avoid having default states for the table cell be obscured (when changing to edit). Basically adding a subview directly is **bad**.

Loading the nib file will register the table view with that specific type of cell

```obj-c
UINib *nib = [UINib nibWithNibName:@"HomepwnerItemCell" bundle:nil];
[[self tableView] registerNib:nib forCellReuseIdentifier:@"HomepwnerItemCell"];
```

Need to do it during `viewDidLoad` since low memory warnings will purge the xib from memory

`UINib` created above reads the XIB file, it loads it into memory and when sent `instantiateWithOwner:options:`, it parses the objects and makes the right connections (the object passed is the File's Owner)

`loadedNibNamed:owner:options:` on `NSBundle` is basically using `UINib` and sends `instantiateWithOwner:options:`.

They both return `NSArray` of the top level objects inside the XIB. When registered with a table view, it looks at the array and looks for the `UITableViewCell`. Only use one instance of `UITableViewCell` for this reason.

## Image manipulation

Could resize, not efficient. Going to create a scaled down version of the image off-screen and keep a pointer to that.

Thumbnails are small enough to archive, `UIImage` doesn't implement the `NSCoding` protocol.

- Solution: get the data as a .png file, throw it in `NSData` and archive that instead

```obj-c
CGSize origImageSize = [image size];
CGRect newRect = CGRectMake(0, 0, 40, 40);
float ratio = MAX(newRect.size.width / origImageSize.width, newRect.size.height / origImageSize.height);
// Start the context drawing offscreen!
UIGraphicsBeginImageContextWithOptions(newRect.size, NO, 0.0);
UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:newRect.size cornerRadius:5.0];
// Clips image to only rounded corners
[path addClip];
CGRect projectRect;
projectRect.size.width = ratio * origImageSize.width;
projectRect.size.height = ratio * origImageSize.height;
projectRect.origin.x = (newRect.size.width - projectRect.size.width) / 2.0;
projectRect.origin.y = (newRect.size.height - projectRect.size.height) / 2.0;
[image drawInRect:projectRect];
UIImage *smallImage = UIGraphicsGetImageFromCurrentImageContext();
[self setThumbnail:smallImage];
NSData *data = UIImagePNGRepresentation(smallImage);
[self setThumbnailData:data];
UIGraphicsEndImageContext();
```

## Relaying actions from UITableViewCells

Want to add a `UIControl/UIButton` that on tap shows a full size image of the thumbnail.

Slaps a UIButton set to custom on top of the thumbnail. Hooked up to a Action Connection.

Problem is that HomepwnerItemCell is handling this message, it has no idea what the model data actually is. View objects here should *only* be view objects.

- Solution: give a pointer to the ItemViewController and send the message to that controller

```obj-c
NSIndexPath *indexPath = [[self tableView] indexPathForCell:self];
NSString *selector = NSStringFromSelector(_cmd);
selector = [selector stringByAppendingString:@"atIndexPath:"];

SEL newSelector = NSSelectorFromString(selector);
[[self controller] performSelector:newSelector withObject:sender withObject:indexPath];
```

## Presenting image in popover controller

```obj-c
CGSize sz = [[self image] size];
[scrollView setContentSize:sz];
[imageView setFrame:CGRectMake(0, 0, sz.width, sz.height)];
[imageView setImage:[self image]];
```

# Core Data

Saving locally? Core data. Saving remotely? Web service/JSON stuff.

The problem with archiving and unarchiving locally is having to read in the entire file for CRUD, Core Data handles the incremental sets of objects that are received and the RAM performance is usually better.

## Object-Relational Mapping

Core Data is essentially an Objective-C ORM to SQLite.

Core Data bring an Objective-C Class as a table and each instance is a row in the database.

Instead of thinking in ORM, this is an **entity-relationship** data pattern.

- Tables and classes are known as *entities*
- Columns/ivars are known as *attributes*
- Attributes/relationships are known as *properties*

Marking one of the attributes as *transient* lets XCode know that we will declare this type at run time instead.

## NSManagedObject and subclasses

Objects fetched from Core Data are by default `NSManagedObject`. This is a subclass of `NSObject` that interacts with the Core Data interface. The managed object has a dictionary that maps properties to values on the entity.

Must declare on the model object in the Core Data file that the specified model subclasses a specific class (ie `BNRItem`)

The objects are marshaled in by Core Data, no need to worry about `init:` for `NSManagedObject` subclasses.

The method we care about here is `awakeFromFetch:`

```obj-c
- (void)awakeFromFetch
{
    [super awakeFromFetch];
    UIImage *tn = [UIImage imageWithData:[self thumbnailData]];
    [self setPrimitiveValue:tn forKey:@"thumbnail"];
}
```

When we want to create an object we will override `awakeFromInsert:` with what we want.

Talking to the DB happens through the `NSManagedObjectContext`. This uses the `NSPersistentStoreCoordinator`.

Initializing the store will involve setting up the `NSManagedObjectContext` and `NSPersistentStoreCoordinator`.

The coordinator answers the following questions:

- What are the entities, their attirubtes and relationships?
- Where do I save and load data from?

The two questions above come from our scaffolded CoreData file that we will shove into `NSManagedObjectModel`.

```obj-c
model = [NSManagedObjectModel mergedModelFromBundles:nil];
NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];

// Write where the SQLite file goes
NSString *path = [self itemArchivePath];
NSURL *storeURL = [NSURL fileURLWithPath:path];
NSError *error = nil;

if (![psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {
    [NSException raise:@"Open failed" format:@"Haha, sucks for you!"];
}

context = [[NSManagedObjectContext alloc] init];
[context setPersistentStoreCoordinator:psc];

// The manager can also undo and roll back, but we won't assign a coordinator for that
[context setUndoManager:nil];
```

Saving simply sends `context:save:` on the `NSManagedObjectContext` and it will throw the error as an arg if it doesn't work out.

### NSFetchRequest & NSPredicate

Grabbing items involves executing a `NSFetchRequest`. Basically a query executor.

```obj-c
NSFetchRequest *req = [[NSFetchRequest alloc] init];
NSEntityDescription *e = [[model entitiesByName] objectForKey:@"BNRItem"];
[req setEntity:e];

NSSortDescriptor *sd = [NSSortDescriptor sortDescriptorWithKey:@"orderingValue" ascending:YES];
[req setSortDescriptors:[NSArray arrayWithObject:sd]];

NSError *error;
NSArray *result = [context executeFetchRequest:req error:&error];
```

When we want only a small subset of all the items we can create a predicate (`where` condition in SQL).

## Adding and deleting items

Adding an item simply calls `NSEntityDescription:insertNewObjectForEntityForName:inManagedObjectContext`.

Removing an item calsl `NSManagedObjectContext:deleteObject:`.

## Reordering items

We set the `orderingValue` attribute to a double so that when we switch two items it simply involves dividing the before and after item by two.

## Mo SQL Mo Problems

Can see executing queries from **Product > Edit Scheme > Run *.app > Arguments** and add `-com.apple.CoreData.SQLDebug => 1`

## Faults

Objects are fetched lazily, getting one object does not get the relationship's objects. Core Data uses *faults*. Fault objects are put in place of the objects until they are needed. It holds the exact query that needs to be executed to get the object.

## Overview

Read *Core Data Programming Guide*, did not delve into...

- `NSFetchRequest` is powerful for what we want from a persistence store. Look at `NSPredicate`, `NSSortOrdering`, `NSExpressionDescription`, and `NSExpression`.
- *Fetched property* is like a hybrid to-many and fetch request.
- Data models change over time, there's an entire book on it
- Validation!
- Can configure per context persistence stores

Technique   | Pros                                                        | Cons
------------|-------------------------------------------------------------|----------------------------------------------------------------------
Archiving   | Allows ordered relationships, easy to deail with versioning | Reads all objects in, no incremental updates (flush the file to disk)
Web service | Easy to share data between devices                          | Requires an internet connection
Core Data   | Lazy fetches, incremental updates                           | Versioning is awkward, need to maintain order ourself

# Localization

**i18n**: making sure hard coded languages aren't in the application
**L10n**: providing appropriate data given Language/Region Format settings

## i18n using NSLocale

Asking the `NSLocale:currentLocale` for the current location in the settings, this returned object can be queried with `objectForKey` with constants to get the proper information (`NSLocaleCurrencySymbol` for instance)

Some objects and methods support locales baked in `NSDateFormatter` has a `locale` property that will use the current set locale.

## Localizing resources

Region specific variables are what we used the `currentLocale` for, but what about other substitutions?

- Images/sounds/interfaces
- String tables that map languages to specific context strings

The file resources go into the application bundle, they are organized by `lproj` directories for each one of the languages. (`en_US` folder where English is the language and US is the region)

`ibtool` is a CLI tool that will help extract all the strings into the proper `.xib` files.

## NSLocalizedString and string tables

String tables are basically just key-value pairs for using associated translations in a different language.

Replace the literals with `NSLocalizedString` where it will take a key to look up and the context of the string, the table is handled by `NSLocalizedString` for us.

```obj-c
NSLocalizedString(@"Sup though", @"A greeting from one person to another");
```

Once we use all localized strings in the applciation, we can generate the associated string table by passing the file to the `genstrings` CLI tool. This will create `Localizable.strings`, putting this into the main project will bundle it up.

# NSUserDefaults

Use the `NSUserDefaults` to store iOS application preferences.

## Using NSUserDefaults

The message `NSUserDefaults:standardUserDefaults` can be used to get the instance for the application. Basically a dictionary with keys as settings and values set to the appropriate option.

You define what each key means in the dictionary, there are *no built-in keys*.

Getting a setting is as simple as `NSUserDefaults:standardUserDefaults:integerForKey:`.

# Touch Events and UIResponder

## Touch Events

There are four methods to override four distinct touch events from views (that subclass `UIResponder`):

Finger/fingers..
- touch the screen: `touchesBegan:withEvent:`
- move across the screen: `touchesMoved:withEvent:`
- removed from the screen: `touchesEnded:withEvent:`

System events (ie incoming call) interrupt touch: `touchesCancelled:withEvent`

When finger touches screen, instantiates a `UITouch`, this instance is in the NSSet of `touches` coming into the method.

- Each instance of `UITouch` belongs to a finger, it lives on as long as the finger is on the screen
- Each view owns the finger for the entire lifecycle (will call every method, even if finger leaves view!)
- Don't need to keep references to `UITouch` instances, the methods will pass in the updated objects

Each time it moves, these events are queued up on `UIApplication`. It almost never fills up. The delivery of the queued touch events go to the proper view that owns the touch via `UIResponder`.

Enabling multi-touch events:

```obj-c
[self setMultipleTouchEnabled:YES];
```

Reminder: refreshing the view needs a quick `UIView:setNeedsDisplay:`.

Detecting double taps through `tapCount`:

```obj-c
for (UITouch *t in touches) {
    if ([t tapCount] > 1) {
        [self clearAll];
        return;
    }
}
```

Packing an object as an addressable value (maybe for a `NSMutableDictionary` key)

```obj-c
NSValue *key = [NSValue valueWithNonretainedObject:t];
```

## The Responder Chain

Other classes subclass `UIResponder`: `UIViewController`, `UIApplication`, and `UIWindow` to name a few.

Non-viewable objects can receive events through the responder chain.

Each responder has a pointer to a `nextResponder` and this is known as the *responder chain*. It will start at the view that was touched and propagate up the delegation chain.

Chain looks something like: `UIView` -> `UIView` (superview) -> `UIViewController` -> `UIWindow` -> `UIApplication`

Can also directly call into the next responder:

```obj-c
[[self nextResponder] touchesBegan:touches withEvent:event];
```

Fun tidbit: `UIControl` is also a `UIResponder` and dispatches the messages based on the type of touches that it receives.

# UIGestureRecognizer and UIMenuController

Patterns of touches are good for taps and drags, if looking for a pinch/swipe we can look at `UIGestureRecognizer`.

## UIGestureRecognizer Subclasses

Don't instantiate ourselves, subclasses handle specific gesture.

Use a target-action pair and attach it to a view, use the message:

```obj-c
- (void)action:(UIGestureRecognizer *)gestureRecognizer;
```

If it's a recognized gesture it will use `action:`, else expect the `touchesBegan:withEvent:`

## Detecting taps with UITapGestureRecognizer

The views will add the gesture recognizer in `initWithFrame:frame`

```obj-c
UITapGestureRecognizer *tapRecognizer = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap:)];
[self addGestureRecognizer:tapRecognizer];
```

The implemented selector from above `tap:` will be sent the instance of `UIGestureRecognizer` for the tap:

```obj-c
- (void)tap:(UIGestureRecognizer *)gr
```

## UIMenuController

The modal popover for menus are a shared singleton across the application. To access it simply call up:

```obj-c
UIMenuController *menu = [UIMenuController sharedMenuController];
UIMenuItem *deleteItem = [[UIMenuItem alloc] initWithTitle:@"Delete" action:@selector(deleteLine:)];
[menu setMenuItems:[NSArray arrayWithObject:deleteItem]];
[menu setTargetRect:CGRectMake(point.x, point.y, 2, 2) inView:self];
```

The presenting view must be the first responder, sent `becomeFirstResponder:`. Must be able to become first responder however by overriding `canBecomeFirstResponder:`.

The menu controller is smart and knows when it should be shown (unimplemented selectors means it won't be shown!).

## UILongPressGestureRecognizer

Long presses have three separate stages: possibly a long press, long press has started, long press has ended

The `state` property is changed to either UIGestureRecognizerStateBegan/Ended.

## UIPanGestureRecognizer and Simultaneous Recognizers

Want to be able to move the line around the screen (panning), time to bust out `UIPanGestureRecognizer`

Up until now, the gesture recognizers have stopped the chain of other recognizers/touch events. However, when we pan we are also long pressing.

Let your view conform to `UIGestureRecognizerDelegate` and set the delegate of a recognizer to the view itself.

Then we can override `gestureRecognizer:shouldRecognizeSimultaneouslyWithGestureRecognizer`.

The pan gesture supports the *changed* state. This is between the *began* and *ended* state for a gesture.

Capturing the translation means calling `gestureRecognizer:translationInView` and at the end making sure the translation point is set to zero again by `gestureRecognizer:setTranslation:inView`.

## UIMenuController and UIResponderStandardEditActions

The `UIMenuController` has all the method dispatching set up for standard edit actions (copy, paste, etc.).

`UIResponder` will implement these methods. They don't do anything. Subclasses like `UITextField` will implement them.

If you implement one of the methods (such as `cut:`) it will show it on the `UIMenuController` automatically. It does this by asking `canPerformAction:withSender` and look for the answer.

## More on UIGestureRecognizer

Gesture recognizers handle all the `UIResponder` methods for you (think `touchesBegan:withEvent:`)

They are greedy and intercept the touch events, need to set properties such as `delaysTouchesBegan`, `delaysTouchesEnded`, `cancelsTouchesInView`.

There are a finite number of states the recognizer can be in:

- Possible
- Began
- Changed
- Ended
- Failed
- Cancelled
- Recognized

There are a few other gestures not covered: `UIPinchGestureRecognizer`, `UISwipeGestureRecognizer`, `UIRotationGestureRecognizer`

# Instruments

## Static Analyzer

Cmd + Shift + B will analyze possible code paths for logical errors and code paths that don't make sense.

For example only one code path to define a variable that is returned will trigger a `Logic Error`.

The analysis can be expanded in the navigator and the code path will show up that has the issue.

## Instruments

Static analysis catches the issues at compile time - what to do for runtime errors?

Instruments! These monitor your application while running and report issues.

- CPU time
- File I/O
- Network I/O

Essentially *profiling* the application.

### Allocations instrument

Displays all objects that are created and the memory dedicated to each one. Click and hold *Run* and click Profile. This will launch the Instruments interface.

The allocations interface allows search per class and will show each instantiated class, memory addresses, the whole shebang.

Looking at a specific instance will show the stack trace on the Extended Detail area. Double clicking the application methods will show a break down of where the memory is allocated.

Use the breadcrumb at the top for navigation.

#### Settings

Stop the profiler and click on the information tag. Choose "Record reference counts".o

#### Heapshot analysis

This can be used in *Generation Analysis* and you can mark heap snapshots while the application is running to see what objects are allocated during a specific time window or between events.

### Time Profiler instrument

Import to basically locate where *time* is being spent. Can prune certain methods under *Specific Data Mining* and drill down to where time is being spent.

Rule of thumb is that some necessary operations are timely (such as redrawing the screen), look around for obvious methods that are wasting time.o

### Leaks Instrument

Useful pre-ARC data, although still possible with retain cycles!

Look at the allocations, under the breadcrumb click *Cycles & Roots* to see potential retain cycles.o

## Xcode Schemes

Workspace: collection of projects

Projects: collection of targets and files

Target: build settings and phases to include reference files

Build: get target, create product that is an application

Scheme contains one or more targets and specifies what to do with it

Scheme > Edit Scheme will show what each target executes as actions (for Run, Profile, Test, etc.)

Option + Scheme action to quickly jump to editing the scheme.

## Build Settings

Build settings that are default for each of the targets

# Core Animation Layer

Need to add QuartzCore to the project.

The two classes are `CALayer` and `CAAnimation`.

`CALayer`: a buffer containing a bitmap, draw to the layer via hardware acceleration.

`CAAnimation`: changes to a layer over time, typically a single property (ie opacity)

## Layers and Views

Images are drawn to a view - uses `CALayer`. Instantiating view creates layer, screen is redrawn and everything in window's hierarchy is redrawn.

Layers created by views are *implicit layers*. This layer hierarchy looks the same as the view hierarchy.

When the views are *rendered*, each of them are *composited* onto the screen. (Respecting each pixels opacity layered on top of each other);

`CALayer` assigns delegate to `UIView` while `UIView` has `layer` property.

The view is an abstraction on top of the visible object and `UIResponder` to tie together interactions. `CALayer` does all the heavy lifting of drawing to the screen.

## Creating a CA Layer

Can be explicitly defined through `layer:alloc:init:` usual construction.

Views set `frame` with `size` and `position`. Layers use `bounds` and `position` where position is the center of the superLayer. (anchorPoint is the relative center and is set to 0.5 0.5).

## Layer Content

Each layer has a bitmap property called `contents`. Can be programmatically set or set to an image.

Drawing is set by subclassing `CALayer` or having the delegate implement the methods.

Notice usage of `CGImage`, `CGColorRef`, both exist on iOS and Mac. (UIKit is only on iOS!).

```obj-c
boxLayer = [[CALayer alloc]  init];
[boxLayer setBounds:CGRectMake(0.0, 0.0, 85.0, 85.0)];
[boxLayer setPosition:CGPointMake(160.0, 100.0)];
UIColor *reddish = [UIColor colorWithRed:1.0 green:0.0 blue:0.0 alpha:0.5];
CGColorRef cgReddish = [reddish CGColor];
[boxLayer setBackgroundColor:cgReddish];
[[self layer] addSublayer:boxLayer];

UIImage *layerImage = [UIImage imageNamed:@"Hypno.png"];
CGImageRef image = [layerImage CGImage];
[boxLayer setContents:(__bridge id)image];
[boxLayer setContentsRect:CGRectMake(-0.1, -0.1, 1.2, 1.2)];
[boxLayer setContentsGravity:kCAGravityResizeAspect];
```

Property on layers has `zPosition` where a greater zPosition is composited *on top* of ones with a lower zPosition.

## Implicitly animated properties

Some properties are animated automatically when their setters are called. (`setPosition:` for example.)

Can appear choppy if the interval less than implicit set interval, must disable implicit animation and use an *animation transaction*. Will batch animations and set parameters.

```obj-c
[CATransaction begin];
[CATransaction setDisableActions:YES];
// do some stuff
[CATransaction commit];
```

## Programmatically generating content

Use delegate method `drawInContext:` that is passed a `CGContextRef` that has the bounding box.

## Layers, Bitmaps, and Contexts

Bitmap: slab of memory that holds RGBA values for each pixel. Sending `setNeedsDisplay:` will send the layer and prepares a new `CGContextRef` that will begin drawing the pixels again.

It will send messages to it's delegate `drawLayer:inContext:` and implicit layers will have this method send `drawRect:`.

# Controlling Animation with CAAnimation

Animation instructions can be added to the `CALayer` instance, they will be queued and animated. Example animation objects could be: `opacity`, `position`, `transform`, etc.

## Animation objects

All animations derive from `CAAnimation`. Basis of timing for anything in a view.

`CAPropertyAnimation` will animate a specific property in a layer.

Abstract superclasses need concrete implementations: `CABasicAnimation` and `CAKeyframeAnimation`.

Basic animations interpolate two values, keyframes will set sspecific values in time that the animation will interpolate. Can change the timing of each keyframe if needed.

Other subclass: `CAAnimationGroup` will concurrently run a set of animations. `CATransition` will animate layers transitioning off and on the screen.

Questions to ask ourselves during animating:

- What type of animation object?
- What key path handles rotation?
- How long should it take?
- What values should it interpolate?

## Timing Functions

Acceleration and deceleration happens with the timing function (instance of `CAMediaTimingFunction`)

Ability to change interpolation with `functionWithName:kCAMediaTimingFunctionEaseInEaseOut`

Animation has a completion handler called `animationDidStop:finished:` that will be called when the animation stops.

Example of creating a bounce animation:

```obj-c
CAKeyframeAnimation *bounce = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
CATransform3D forward = CATransform3DMakeScale(1.3, 1.3, 1.0);
CATransform3D back = CATransform3DMakeScale(0.7, 0.7, 1.0);
CATransform3D forward2 = CATransform3DMakeScale(1.2, 1.2, 1.0);
CATransform3D back2 = CATransform3DMakeScale(0.9, 0.9, 1.0);
[bounce setValues:[NSArray arrayWithObjects:[NSValue valueWithCATransform3D:CATransform3DIdentity],
                   [NSValue valueWithCATransform3D:forward],
                   [NSValue valueWithCATransform3D:back],
                   [NSValue valueWithCATransform3D:forward2],
                   [NSValue valueWithCATransform3D:back2],
                   nil]];
[bounce setDuration:0.6];
[[timeLabel layer] addAnimation:bounce forKey:@"bounceAnimation"];
```

# UIStoryBoard

Lays out relationships between view controllers, similar to `.xib` files.

Can add `ViewControllers to the navigationController by `Ctrl+drag` and setting the relationship to `rootViewController`.

## Segues

Setting up interactions to go between different controllers

Going between to view controllers is represented by an instance of `UIStoryboardSegue`.

Has a...

- Style: how the view controller is presented (pushed, modal, etc.)
- Action item: view object that triggered the segue (ie UIButton)
- Identifier: programmatic access to segue (came from shake or action with no action item)

Modal view controllers can be dismissed by subclassing `UIViewController` and overriding an `IBAction` method to get rid of the `presentingViewController`. Note that this must be done on the storyboard by `ctrl+drag` from the button/action item to the toolbar at the bottom for the view controller.


## More on Storyboards

Storyboards replace need for code that does this...

```obj-c
UIViewController *vc = [[UIViewController alloc] init];
[[self navigationController] pushViewController:vc];
```

Pros:

- Easily show flows to client/coworker
- Remove simple boilerplate code
- Pretty
- Sttic content easily

Cons:

- Collaboration on a team is hard
- Version control? Good luck!
- Context switch from programming to using a UI (still need to jump in to override segue transitions usually)
- Flexibility: more work for advanced things
- New ViewControllers are instantiated for segues, we can handle this better in code

# Web Services and UIWebView

## Web Services

Can leverage use of HTTP and embedding the web services into your application that will communicate over HTTP and handle JSON/XML responses.

## NSURL, NSURLRequest, and NSURLConnection

NSURLConnection will have a request (NSURLRequest) which will have a URL (NSURL).

- `NSURL`: location of web resource, this is usually the base address
- `NSURLRequest`: holds data to talk to server (caching, TTL, and other HTTP protocol)
  - there's a subclass known as `NSMutableURLRequest`.
- `NSURLConnection`: makes the connection to the web server and will handle response

### Formatting URLs and requests

Generally the URLs will look something like...

```
http://baseURL.com/serviceName?argumentX=valueX&argumentY=valueY
```

### Working with NSURLConnection

Connection is either sync/async.

Needs location (URL), data to pass (NSURLRequest), and a delegate once the response is returned.

```obj-c
NSURL *url = [NSURL URLWithString:@"http://forums.bignerdranch.com/smartfeed.php?limit=1_DAY&sort_by=standard&feed_type=RSS2.0&feed_style=COMPACT"];
NSURLRequest *req = [NSURLRequest requestWithURL:url];
connection = [[NSURLConnection alloc] initWithRequest:req delegate:self];
```

Lifecycle of `NSURLConnection`:

- Delegate will send `start:`
- Connection will send `connection:didReceiveResponse:`
- Connection will send (multiple times possibly) `connection:didReceiveData:`
- Connection will end with `connection:didFinishLoading`

The delegate conforms to the protocol for NSURLConnectionDelegate. If all goes well the above lifecycle will happen otherwise the message `connection:didFailWithError:` will be sent to the delegate. It happens only during connection failure or the server doesn't exist, HTTP status code failures (4xx) are sent with `connection:didReceiveData:`.

## Parsing XML with NSXMLParser

Takes a chunk of XML data and finds the selector or element you are looking for.

```obj-c
NSXMLParser *parser = [[NSXMLParser alloc] initWithData:xmlData];
[parser setDelegate:self];
[parser parse];
```

- Parser finds start of `channel` element, create a `RSSChannel`
- Parser finds `title` or `description` and is inside `channel`, set the properties on the instinace
- Finds an `item` element, create an instance of `RSSItem` and add it to the `items` on the channel
- Finds `title` or `link` and is inside an `item` element, set the properties on the instance

The parser is stateless and thus to handle which elements are assigned properties we change the `delegate` property during parsing.

Classes conform to `NSXMLParserDelegate` and will look for `parser:didStartElement:`.

## UIWebView

Instead of jumping out into Safari to open a web link, we can open it in the application using a `UIWebView`.

```obj-c
// In a UIViewController class we override loadView...
CGRect screenFrame = [[UIScreen mainScreen] applicationFrame];
UIWebView *wv = [[UIWebView alloc] initWithFrame:screenFrame];
[wv setScalesPageToFit:YES];
[self setView: wv];
```

We can have the view that is handling the web view being brought into view basically push the request onto the stack through the navigation controller.

```obj-c
[[self navigationController] pushViewController:webViewController animated:YES];
RSSItem *entry = [[channel items] objectAtIndex:[indexPath row]];
NSURL *url = [NSURL URLWithString:[entry link]];
NSURLRequest *req = [NSURLRequest requestWithURL:url];
[[webViewController webView] loadRequest:req];
[[webViewController navigationItem] setTitle:[entry title]];
```

# UISplitViewController and NSRegularExpression

iPads can take advantage of `UISplitViewController`. Initialization similar to `UITabViewController` but instead of an array of `UIViewController`s it will only accept two: a master view controller and detail view controller.

```obj-c
UINavigationController *detailNav = [[UINavigationController alloc] initWithRootViewController:wvc];
NSArray *vcs = [NSArray arrayWithObjects:unc, detailNav, nil];
UISplitViewController *svc = [[UISplitViewController alloc] init];
[svc setDelegate:wvc];
[svc setViewControllers:vcs];
[[self window] setRootViewController:svc];
```

Check that the `uiViewController:splitViewController:` is not declared, if not it will handle itself and the selection (`tableView:didSelectRowAtIndexPath:`) will not need to support pushing the webview.

## Master-Detail Communication

Simple applications we passed around pointers (NerdFeed passed the `WebViewController` around

```obj-c
@protocol ListViewControllerDelegate
- (void)listViewController:(ListViewController *)lvc handleObject:(id)object;
@end
```

# Blocks

Set of instructions.

Technically an object, called similar to a function.

Basically: closures/anonymous functions.

## Blocks & Block Syntax

Objective-C: classes that have methods that the class implements. Different than a function syntactically.

Difference between function/method: method executed in context of class.

Blocks bridge the two, method or function, can pass executable context. Can also act like an object.

## Declaring block variables

Block is an object, variables can point to them!

Block variables have a name and a type, identifies return values and arguments.

```obj-c
// Takes two ints, returns an int
int (^adder)(int a, int b);
```

Above the name of the variable is `adder`. Points to a function that takes two `int`s and returns an `int`. The `^` says that this is a pointer to a block, think of it as `*` for blocks.

Takes nothing, returns nothing:

```obj-c
void (^foo)(void);
```

The above simply declares that this variable is of this type of block, a *block literal* is what we see below:

```obj-c
^int(int x, int y) {
  return x + y;
}
```

Combining what we've learned, we can assign the declaration to the literal and have the execution context be a function call:

```obj-c
int(^foo)(int, int) = ^int(int x, int y) {
  return x + y;
};

foo(1,2);
```

- Block variable does not need to declare argument names
- Block literal needs to declare argument names

## Basics of using blocks

Declaring a function prototype in the header files where a block in an argument has the following syntax:

```obj-c
- (void)setEquation:(int (^)(int, int))block;
```

## Variable capturing

Basically just a closure. Captures local state.

Note: the blocks capture the closure at time of declaration, changing the value of a capture variable does *not* change the value in the closure.

```obj-c
int multiplier = 3;
[executor setEquation:^int(int x, int y) {
  return x * multiplier;
}];
multiplier = 4;

// multiplier is always going to be 3
```

Each application has a *main operation queue*, adding operations will queue the events that have their own *run loop*.

```obj-c
[[NSOperationQueue mainQueue] addOperationWithBlock:^void(void) {
  NSLog(@"%d", [executor computeWithValue:2 andValue:5]);
}];
```

Note that above even though it was queued and ran after `application:didFinishLaunchingWithOptions:` we have to be aware that the block is actually retaining a *strong reference* to the variables.

## Typical Block Usage

Blocks are mostly used for callbacks. When events finish, call this chunk of code. These are known in UI events as *completion blocks*.

Simpler than setting up a delegate or target-action pair, there are no objects to send messages to!

## __block modifier, abbreviated syntax, memory

Blocks know their return types, we can only declare the argument types that are passed in:

```obj-c
int (^block)(void) = ^(void) {
  return 10;
};
```

We can also say that if it doesn't take any arguments, let's leave those out of the literal:

```obj-c
void (^block)(void) = ^{
  // yeah;
}
```

Variables usually can't be mutated that are declared outside the scope of a block, we can add the `__block` modifier to get around this...

```obj-c
__block int count = 0;

void (^block)(void) = ^{
  count++;
};
```

Blocks are created on the stack, to move to the heap we need to pass the `copy` message.

## Pros and cons of different callback designs

Target-action useful when...

- Two actions are coupled (view controller and view)

Delegation useful when....

- Object receives a lot of events and we want a protocol to formalize this for us

Notification useful when...

- Multiple objects want to be able to respond

Blocks are not object related, useful when it's going to happen once or a quick task. No need to assign ownership for trivial functionality to an object that it may not belong on.

# Model-View-Controller-Store

Remember Homepwner and the custom item stores we built on top of our iOS app!

## The Need for Stores

Mobile applications usually make use of external sources of data. (servers, filesystem, location hardware, camera, etc).

*Request logic*: code that fetches from an external source

Request logic exists in `NSFetchRequest` for Core Data, `NSURLRequest` for `NSURLConnection`, etc.

In traditional MVC, request logic is in the controller. Not optimal. Muddies the waters. What is a controller really even suppsoed to do?

- Traditional MVC, a few controllers talking to a few models is fine
- Messy MVC, start pulling data from CoreData, servers, etc.
- Controllers shouldn't care about this!

Throw the request logic into a store, minimizes the interaction the controller has to have and delegates the responsibility for all fetching of data to a single object.

## Designing a store object

- What external sources?
- Singleton?
- Deliver results of request?

### Determining external source

Questions to ask here...

- 1-1 with the external source?
- If no to the above, break into multiple stores?

### Determining singleton status

Is there state that needs to persist for a store?

Example: `CLLocationManager` is essentially a store, needs to tune parameters to get an instance of the store however

# iCloud

iCloud is key-value storage and document storage *in the cloud*.

Key-value is for preference-like data across devices. `NSUbiquitousKeyValueStore`, similar API to `NSUserDefaults`.

The document storage can be used for anything, docs/spread sheets/plain text etc.

Used classes are...

- `UIDocument`
- `NSFileCoordinator`
- `NSMetadataQuery`

The protocol they all conform to are `NSFilePresenter`.

## Ubiquity Containers

Usually the file is tied to the application sandbox, iCloud changes up the game with directories called: *ubiquity containers* (so fancy.)

- Uniquely identified by the *container identifier string*
- Must add the container identifier to the list of *entitlements*

Find the *Entitlements* section in the target and enable it.

Default is the application bundle name (`com.bignerdranch.Nerdfeed` here).

The ubiquity container needs to be the same across all the different devices.

CoreData stores the transaction log file in the cloud. When changes occur the transaction file is loaded and downlaoded.

We set the `NSPersistentStoreUbiquitousContentNameKey` and the `NSPersistentStoreUbiquitousContentURLKey` for the `NSPersistentStoreCoordinator`s options.
