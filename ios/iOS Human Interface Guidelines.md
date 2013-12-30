## Designing for iOS 7

- Deference: help users interact, don't compete
- Clarity: text legible, icons clear, adornments subtle, focus on functionality
- Depth: layers and motion show life and give users enjoyment

Consider this workflow when redesigning:

- Take out UI, look at core functionality (is it worthwhile?)
- Use iOS 7 components going in, nothing gratutiously
- Defy precedent, question assumptions, focus on content

### Defer to content

Users content at the center of an application.

- Take advantage of the whole screen
- Reconsider visual indictators for the physicalworld (bezels, gradients, drop shadows)
- Translucent UI should hint at content behind

### Provide clarity

Make sure content is there to consume as *clearly* as possible

- Use negative space, easier to read - calms the user
- Use color, for states and interactivity. Promote visual consistency
- Ensure legibility, dynamically sized fonts should still make the app accessible
- Borderless buttons: use context, color, and CTA title

### Use depth to communicate

Content in hierarchy/position, help use understand relationship

*Translucency* can provide visual hierarchy (floating above)

Actually show layered elements - think Reminders app

Enhance transitions to show moving through a relationship - think Calendar app zoom in/out for time

## iOS App Anatomy

Top to bottom...

| Element           |
| ----------------- |
| Navigation bar    |
| Segmented control |
| Views/alerts      |
| Toolbar buttons   |
| Tab bar           |

UIKit separates elements into four categories:

- Bars: contextual *navigation* information for where the user is
- Content views: app-specific content, triggers edit states
- Controls: perform actions, display information
- Temporary view: show important information, make the user make a choice

In additional to elements there are other interaction dimensions: gestures, drawing, accessiblity

Developers think of view/view controllers, users see *screens*, visual state or mode in an application

## Starting and Stopping

### Start instantly

Superior experience shows useful content immediately.

*Avoid splash screens or other experiences that don't take the user to content*

**No setup information**:

- Focus on the 80% of users, let the app behave as that user would expect who supplies no information
- Get informatoin from other sources: built in app or other device settings
- Prompt the information to be entered within the application, NOT settings!

**Delay requiring to log in**

**Launch in default orientation**

- This is usually portrait, if it is only landscape then launch into that

**Launch image closely resembles first screen**: give the users an impression that the applicatoin is *fast*

**No EULA**

**Restore where the user left off when started**

### Always Be Prepared to Stop

**There's no close or quit button, ever.**

**Save user data fast and often**: this will happen at any time, the application needs to expect this

When the user gets deep, save where they are in the application so that they don't need to navigate again.

Don't programmatically quit the application, users interpret this as a **crash**

- Display an 'Unavailable' screen
- Alert when they try to use a non-functional part of the application

### Layout

More than how it *looks*, show the user...

- What's important?
- What are their choices?
- How do things relate?

**Make it easy to focus on the content**:

- The upper left is the most important, the bottom right is the least

**Visual weight to balance important**:

- Large things should catch the eye
- Colorful things should catch the eye
- Combination of the two are the CTA for a view

**Use alignment**:

- Communicates hierarchy
- Visual grouping

Default views should **provide primary content**

Brace for changes in text size

## Navigation

Three main styles:

- Hierarchical: one choice per screen (Settings)
- Flat: users navigate to category (tables)
- Content/experience driven

There are a few things to note when the user navigates around your application...

- Users should know *where they are* and how to get to *where they are going*
- Use navigation bars when traversing a hierarchy of data
- Tab bar when peer categories are available
- Use a page control when multiple *instances* are available (Weather app cities)
- Golden rule: **One way to get to a screen**

Additional components

- Segmented control: different "aspects" of the screen, not necessarily navigation
- Toolbar: act on content on the screen (similar to tab bar without navigation)

## Modal Contexts

**Modality**: a mode which something exists or is experienced

Good: Allows the action to be performed without interaction

Bad: closes off interaction from rest of the application

People interact in *nonlinear ways*, keep this in mind when designing modal experiences.

Use it when:

- You NEED the user's attention
- MUST be completed/abandoned, potentially ambiguous data otherwise

Keep in mind:

- **Keep modal tasks simple, short, and focused**: no subtasks or embedded applications, clear navigation to completing task
- **Use a done button at the last possible moment**: gather all necessary data first
- **Alerts are for actionable information**: the alarm is going off, snooze or turn off?
- **Respect notification settings**

## Interactivity and Feedback

### Users know the Standard Gestures

Users expect gestures to work similarly in all applications they use

- Tap: select/control an item
- Drag: scroll/pan, drag an element
- Flick: scroll/pan quickly
- Swipe: one finger (return to previous/reveal hidden view/delete button)
- Double tap: zoom in, center block of content, zoom out otherwise
- Pinch: open to zoom in, close to zoom out
- Touch and hold: editable/selectable text, display magnified view
- Shake: undo/redo action

There are system-wide gestures that users expect to *always* work:

- **Don't associate different actions to a gesture**: if it's not a game, leave it be
- **Don't create custom gestures for existing actions**: use the gesture that the user expects
- **Gestures expedite tasks**: they aren't the only way to perform it!

### Interactive Elements Invite Touch

Interactivity comes from cues such as color, location, context,icons/labels

**Don't decorate anything to provide emphasis**

- Colors: buttons show when something can be pressed
- Back button: location indicates control over navigation
- Clear title: "Add Bookmark" makes embellishment superfluous

**Button border/background when necessary**: users know what is interactive in bars/tables/alerts. Content area may be ambiguous, exercise with *caution*.

### Feedback Aids Understanding

Provides users signals for: what it's doing, what they can do next, understand results

- **Integrate status into UI**: think, number of unread messages in Mail tab bar
- **Avoid alerts**: essential actionable information only

### Inputting Information Should Be Easy

Asking for lots of input slows users down, only get what you need!

- **Make choice easy**: what day is it? Show it in a table view!
- **Get information from the system**: enter your contacts? NOPE.
- **Balance request for input for something in return**: elegantly show progress

## Animation

Animation can provide:

- Status/feedback
- Enhance direct manipulation
- Visualize results

Keep in mind:

- **Cautiously add animation**: don't obstruct flow, decrease performance, distract users
- **Custom animation?**: better do the physics right
- **Consistent animations**: use it the same throughout the app

## Branding

More than assets, it is a *unique and delightful experience*

- Custom elements should function by themselves
- Doesn't need to look built in, explore deference/clarity/depth yourself

Keep in mind:

- **Unobstrusive assets**: seem subtle, not an advertisement
- **Respect the space**: remember whitespace, just because it's there doesn't mean shove a logo there

## Color and Typography

### Color Enhances Communication

Color...

- Indiciate interactivity
- Is liveliness
- Is visual continuity

Keep in mind:

- **Multiple colors work together**: choose the palette carefully
- **Notice contrast in different contexts**: think of the colors in the nav bar, tab bar, content area, etc.
- **Custom bar tints**: the translucency will jump in, remember that
- **Note color blindness**: make sure to show state in another way
- **Key color**: choose a color to represent interactivity
- **Don't mix up the key color**
- **Color perception is culture-dependent**: make sure the appropriate message is delivered

**Don't distract with color.**

### Text Should Always Be Legible

Dynamic typing in iOS 7:

- Automatically adjusts spacing/height
- Specifies semantic context styles (body, footer, etc.)
- Respects accessibility settings

Keep in mind:

- **Prioritize size**: not everything needs to be larger, think of the content the user wants to see
- **Adjust layouts**: depending on text size can provide different column layouts
- **All styles of custom font are legible**: period.
- **One font**: don't need to get crazy here, choose one and be done.

## Icons and Graphics

### App Icon

Initial opinion is based on the app's icon. First impressions matter.

- This is a part of your brand.
- Be unique. Uncluttered. Engaging. Memorable.
- Must look good at all sizes and backgrounds

### Bar Icons

Stock small icons for toolbars, tab bars, nav bars, etc. They are a good idea.

Useful text could trump any icon. Think "Today" in the Calendar app. There's no possibly useful icon for that.

### Graphics

Apps are graphically rich:

- **Support Retina**: give the @2x assets for all the things.
- **Display original**: don't mess around with zooming or scaling

## Terminology and Wording

This is an ongoing conversation with users. Help the user feel comfortable.

No jargon.

- **Your users need to understand**: how else will they use your application?
- **Friendly tone**: don't be too familiar, note they are reading this all the time
- **Newspaper editor**: watch out for redundant words, be concise
- **Short labels**: or good icons, users are looking at a glance
- **Be accurate with dates**: today/tomorrow, consider locale
- **App Store description**: this matters...yeah.

## Integrating with iOS

Seem at home with iOS, don't copy anything.

### Use Standard UI Elements Correctly

Prefer UIKit elements versus custom elements:

- Redesigned elements will be automatically updated
- Standard elements provide methods to customize appearance and/or behavior
- Users are more familiar with them

When using standard UI elements:

- **Follow guidelines**: don’t deviate from how the user expects the elements to be used in other applications
- Don’t mix element styles, stick to iOS 6/7/etc through standard usage
- **Don’t customize what is standard**: if UIKit offers it, don’t put a new element there
- **Don’t override system defined buttons/icons**: tie what the system provides to it’s actual value
- **Immersive tasks/experiences**: this is when to bust out the custom

### Respond to Changes in Device Orientation

Respond appropriately when the user changes orientation, specifically

- Maintain focus on content
- Optimize for all orientations
- Launch in supported orientation (such as games)
- **Never** tell the user they must rotate

Users typically go to landscape to *see more*.

### Downplay File and Documents

The user shouldn’t think about the filesystem on an iOS device.

Defer to iCloud when management of documents is needed.

- The only exception is document creation applications, obviously.
	- Should be graphical
	- Ideally gesture based
	- Include *new document* as a feature

### Be Configurable If Necessary

Let the user change their settings in app, don’t send them off to the Settings application

### Take Advantage of iOS Technologies

Lots of system supported technology is available for the user to take advantage of:

- Multitasking
- VoiceOver
- PassBook
- In-App Purchases (IAP)
- In-App Advertising (iAd)
- Game Center

## Design Principles

### Aesthetic Integrity

How well does the application integrate appearance with functionality, is it coherent?

Examples:

- A user performs a serious task: keep decoration and use standard UI elements
- A user is in a game: captivate their eyes with creative appearance

### Consistency

Allows for knowledge transfer between different screens in the application.

Don’t copy other applications, is not stylistically stagnant

Ask yourself these questions with respect to design consistency:

- Consistent with iOS? System controls, views, icons correct?
- Consistent with itself? Are fonts/icons/UI elements the same from view to view?
- Consistent with previous versions of the application?

### Direct Manipulation

Get the user engaged in a task with clearly displayed results.

### Feedback

Show the user results, update progress on the screen.

Every built in element responds in some way to a user action.

**Do not rely on sound.**

### Metaphors

Metaphors with familiar experiences in real/digital worlds are useful.

- Moving layered views to show what’s underneath
- Drag/flick/swipe objects in a game
- Tap switch, slide slider, spin picker
- Flip through book

### User Control

*People* initiate control and actions.

Do not take decisions for the user, let them feel like they are in control. Facilitates a more predictable experience.

## From Concept to Product

### Define Your App

Have an **app definition statement**: main purpose and audience.

Helps prune features, get the right behaviors down.

1. List all features: enumerate all primary/secondary/tertiary features
2. Determine who the users are: three characteristics that describe the target audience
3. Filter enumerated features through audience definition

The app definition statement will help when considering:

- Adding a new feature
- Editing look and behavior of the UI
- Terminology used in app

### Tailor Customization to the Task

*Balance UI customization with clarity and ease of use.*

Consider customization **early**. Need to stay focused on UI, branding/originality/marketability is secondary.

How often do users perform the task, and under what circumstances?

- **Always have a reason to customize**: enhance the experience
- **Avoid cognitive burden**: familiarity is nice, facilitate ease of use
- **Internal consistency**: users rely from other parts of your app
- **Defer to content**: the UI doesn’t compete, let the content shine
- **Test all custom UI elements**

### Prototype and Iterate

Create prototypes for user testing. Benefit from the fresh perspective.

Paper prototype/wireframes are fine to start, map out the flow.

Fleshed out prototypes that run on device will show the funky parts of the application.

Storyboards are great for this. Just provide enough context to emulate a *real experience*.

## Web Content in iOS

Crisp text and sharp images with rotation is key for websites, no plugins - the website should be supported.

Look out for:

- Setting the viewport correctly
- No `fixed` positioning for CSS, zoom/pan is still possible
- Touch UI, don't rely that the user has a mouse

When designing the experience, remember that:

- The view should support keyboard and form control
- Pop-up menus are native elements on iOS

## Running on iPhone 5

**176 pixels taller**, this is equivalent to two more standard table rows.

Use components with auto-layout for cross phone compatibility.

More content will be displayed, don’t set *any static heights* on the main UI view.

Stretch background areas. The vertical space will be utilized on iPhone 5!

**Static positioning is bad**: if it is used, consider re-centering the dominant visual elements.

Don’t use it to show a banner or a new button.

# iOS Technologies

## Passbook

Manage passes: boarding passes, coupons, membership cards, and tickets.

Assemble passes, distribute to users and update later.

To make a great passbook app:

- **Avoid mimicking physical pass**: design aesthetic to look like it belongs on iOS7 - simple and clean
- **Important information on front of pass**: be selective.
- **Use solid colors**: white looks ugly
- **Logo text field is your company name**: no custom fonts here though
- **Company white logo**: “engraving” is 1 px y offset, 1 px blur, 35% opacity
- **Rectangular barcode!**: square barcode will create an empty gutter
- **Smallest image is desirable**
- **Enhance utility of pass**: show real time information (i.e. boarding pass)

## Multitasking

Apps can go into suspended state, on switching back it won’t reload the UI.

The *multitasking UI* is when choosing an application from the double home button tap, we can change that UI.

This means we should:

- Handle interruptions and audio gracefully
- Transition to and from background smoothly
- Behave respectfully when not in foreground

Some things to remember:

- **Prepare for interruptions**
- **Make sure the double high status bar is supported**:  happens during calls
- **Pause activities**: user doesn’t want to miss any content
- **Audio behaves**: people have expectations when interruptions happen (main track becomes dulled)
- **Use notifications sparingly**
- **Finish tasks in background**: expect the user to leave your app

## Routing

Maps app provides *directions*, routing apps should provide *transit information*.

- Confirm expectations that the application will provide everything the user wants:
	- Define regions the application will support
	- Define the types of transit
- Streamline UI: the users are on the go!
- Focus on route, step by step
- Re-entering information is **bad**, take the route from Maps!
- Transit information is both *graphical* (map view) and *textual* (directions)
- Enrich with additional information (i.e. **pins**)
- Different transit options: walk, weather, day, etc are all factors and ways to sort
- Live *updates* on the route if there are any delays etc.


## Social Media

Integration for sharing channels on social media.

- **Allow users to post without leaving your app**: look into `SLComposeViewController` class to integrate the Social Framework
- **Avoid asking for log in**: using SSO we get authorization without asking to reauthenticate
- **Activity view controllers to facilitate decisions**: let the user choose which service they want to use

## iCloud

Examine closely how user-created content is treated in *your* application.

**Transparency** is the reason users use iCloud, they aren’t aware of where their content is or what version they are looking at

- Make it easy to enable iCloud
- The users have a quote of space, respect that
- Don’t ask the users what to store in iCloud, the application context is what they expect
- Determine whether it’s key-value storage for preferences or documents that users have created
- Application needs to work when iCloud isn’t available
- Don’t allow local vs iCloud documents: encourage _pervasive availability_
- Automatically sync content
- Warn users what it means to delete a document
- Show users document conflicts unobtrusively: let them differentiate between versions easily

## In-App Purchases (IAP)

Users can purchase content when using your application, this can happen when:

- Upgrade to a premium version
- Subscribe to content
- New virtual items
- Buy and download new content

When building out in-app purchases, iOS only provides the framework to make the purchase, the rest of the business logic lies on the application.

Remember to…

- Integrate store experience: let the IAP feel like home in your application
- Use simple succinct descriptions of what the user is buying
- The default confirmation alert should not be changed

## Game Center

GameKit API provides leader boards and ways to find other players with the users contacts.

Remember to…

- Not create custom UI to sign into Game Center
- Let users turn off voice chat

## Notification Center

Lets users check their notifications in one spot.

**Local notification**: scheduled by app (due dates for tasks, etc)

**Push notification**: sent to APNS and goes to all devices that the user is signed onto

Be aware of the different notification styles your notification can appear in (Banner, Alert, Badge, etc).

- Banner: translucent onscreen view at the top that goes away
- Alert: persistent onscreen notification
- Badge: oval for pending notifications for the application

Don’t send multiple notifications for the same event.

Custom messaging should _not_ include the application name.

##  iAd Rich Media Ads

Revenue generation for ads!

Served by the iAd network, all you need to do in your application is provide the view.

Couple of different sizes: (standard, medium rectangle, full screen)

- Standard: takes up banner across the screen, pretty small
- Medium rectangle: similar to standard banner, available only on iPads to go across the screen
- Full screen: most of the screen belongs to the ad (can be modal or page)

Some guidelines:

- Standard banners go on the bottom: always above the most bottom element (tab view, toolbar, etc)
- Medium rectangles should not interfere with content: usually best n the bottom out of peoples way
- Full screen: present modally during UX interludes, when transitioning show non modally
- Display in both orientations

## AirPrint

You can print stuff from the share bar.

## Location Services

Help the users feel comfortable since location services can sometimes be creepy:

- Let them understand _why_ the location is needed
- Ask permission at startup **if and only if** the application can’t function without it
- Only ask when the actual feature that uses the data is triggered
- Check _Location Services preference_ before directly asking the user

## Quick Look

Allows for in-application previews. They can see some document data and it will be opened in a new view or modal view.

- **iPad display preview modally**
- **iPhone display in dedicated navigable view**

## Sound

### Understand User Expectation

The user switched the device to silent because…

- Don’t want to be interrupted
- Avoid hearing sounds from user actions
- Avoid listening to game sounds and soundtracks

Certain contexts make sense to override the use of the silence button (Media playback, alarm clock, sound clip in language application, audio chat application)

### Define audio behavior of the application

Total volume govern by system setting, can mix and adjust independent volume channels in application

Application can display audio route picker (device to headphones, or speakers, etc.)

The system-provided slider has been made available when the user needs to adjust the volume.

Audio sessions can be used to express how audio on device responds to output configurations. (solo ambient, ambient, playback, record, play and record, and audio processing, etc)

###  Manage audio interruptions

iOS expects the application to:

- Identify interruptions that can be caused
- How to respond after an audio interruption ends

## VoiceOver

Accessibility for low vision and blind users.

Can help increase the user base and address the accessibility concerns of governing bodies who try to adopt your application.

## Edit Menu

The little popover that has Cut/Paste/Select etc.

The behaviors can be adjusted given the content of the application.

- Specify which are appropriate
- Position of the menu
- Define “selectable” elements when the user double taps

To have the standard behavior, be sure that:

- Commands make sense in the context
- The menu makes sense in all layouts on the screen
- Double tap and touch and hold gestures are supported
- Don’t create buttons that exist in the edit menu
- Enable selection of static text if available
- Don’t make buttons selectable
- Undo/redo needs to be available with copy and paste
- Custom actions come *after* system provided ones

##  Undo and Redo

Users initiate Undo by shaking the device

- Undo what they just typed
- Redo previously undone typing
- Cancel undo operation

The application can specify:

- Actions to undo
- What level of shake means undo
- Number of levels to support for undo

*Don’t overload the shake gesture*

## Keyboard and Input Views

Customized keyboards are available. Click sound must be the same as the system enabled keyboard.

# UI Elements

## Bars

### Status Bar

The status bar shows device data and environment:

- Transparent
- Always appears at top

Remember…

- _No customization_: users rely on consistency of the system-provided status bar
- _No content behind the bar_: there should only be a consistent 
- _Hide status bar appropriately_: when in full screen
- _Use network activity_: turn on the spiny thing when things are happening

### Navigation bar

Shows information hierarchy

The actual bar at the top:

- Is translucent
- Appears at the top right below (on iPhone)

The content displayed belongs in a _hierarchy_, if it is not then we should consider a toolbar.

- The bar’s navigation title should update when moving to a new view
- The back button should be labeled with the previous title

Remember…

- Prompt the user with what data they want to see (search bar) when moving into a navigation view
- When customizing, make _consistent_ throughout the application

### Toolbar

Perform actions at the top of a view that have no navigation context

This toolbar is…

- Translucent
- At the bottom edge of a view on iPhone

Remember to…

- Use comments that make sense in the context (view dependent!)
- Use a segmented control to provide new modes/perspectives
- Icons for more than 3 actions
- Text titled buttons need to be spaced out

### Toolbar and Navigation Bar Buttons

Used in the above views for bar icons.

Can be tinted with the `tintColor` property.

List of example actions: share, camera, compose, bookmarks, search, add, trash, organize, reply, refresh, play, fast forward, pause, rewind

### Tab Bar

Switch between subtasks, views, modes

- It’s translucent
- Appears at the bottom of the screen
- Maximum of 5 items
- Always the same height
- Can display badges for each tab icon

Remember this bar is used for…

- Organizing information at the top level
- Don’t remove the tabs
- The tab button _only_ changes views
- Badging can be used to unobtrusively communicate

There are some default tab bar icons:

- Bookmarks
- Contacts
- Downloads
- Favorites
- Featured
- History
- Most Recent
- Most Viewed
- Recents
- Search
- Top Rated

###  Search Bar

Used for searching

- Can have placeholder text
- Provide bookmarks to facilitate search
- Provide clear to wipe the field
- Results list icon can show that there are results
- Prompt when there is a context for search

Two styles: _prominent_ (Mail) and _minimal_ (Music)

### Scope Bar

This is available at the top to define a scope for search (All/Devices/…)

## Content Views

### Activity

An activity is a system provided or custom service available through activity view controller

- Object representing an *action* that the app can perform
- Icon looks similar to a bar button icon

### Activity View Controller

Provides transient view for system provided or custom services

- Displays list of services the user can use to perform on the content
- Appears in an action sheet on iPhone

### Collection View

Shows a managed collection of items, customizable layout

- Distinguish subsets of items
- Support animated transitions between layouts

Don’t distract the user, this should be enhancing their task at hand…

- Don’t use it when a table view would be better
- Make it easy to select an item
- Dynamic layouts are dangerous

### Container View Controller

Presents child views (tab tab view, navigation view, split view, etc.)

### Image View

Shows an image

- No defined UX by default
- Properties to stretch, scale, etc. are available

- Ensure images are the same size and scale for all viewed images

### Map View

Shows a geographic area

- Can display annotations
- Support for programmatic zooming and panning

Remember for map views that:

- Users interact with the map themselves
- The pin colors should be consistent

### Scroll View

Lets users see outside of the bounding view

- No predefined UX
- Flashes scrollers when first viewed
- Responds to acceleration of gestures
- Paging mode

Remember that…

- Zoom behavior should be appropriate: pinch, double tap, etc.
- Page controls useful when the content is paginated
- Only one scroll view at a time

### Table View

Data in a _single_ column, controls to add/delete rows

Couple of styles:

- Plain: each row added with a label and icon
- Grouped: separated into groups

Various elements that could extend the view of a row:

- Checkmark
- Disclosure (>)
- Detail disclosure (i)
- Row reorder
- Row insert
- Delete control (-)
- Delete button

There are various cell styles that a row could be rendered in:

- Default: icon/title
- Subtitle: icon/title/subtitle
- Value 1: icon/title/right label
- Value 2: title/label

Remember when using table views…

- Feedback when an item is selected
- Do not wait until all data is available: show onscreen immediately
- Succinct text: truncation is the worst case scenario

### Text View

- Rectangle of any height
- Supports scrolling when out of bounds
- Supports custom fonts and colors

### Web View

Displays HTML content

- Automation conversion on iOS for fields (phone numbers, dates, addresses, etc.)
- Make it seem native it is not a mini browser within the application

## Controls

### Activity Indicator

Shows a task in progress

- Disappears when task is completed
- Doesn’t allow for additional user interaction

### Contact Add Button

Adds a contact to an existing text view or field on the screen

- Display the contacts
- Facilitate the user interaction to add the contact

### Date Picker

Picker for the date, supports up to _4_ different wheels

- Support dark dark for current value
- Date/time
- Time
- Date
- Countdown timer

### Detail Disclosure

Support view for when additional details need to be displayed for the item

### Info Button

Reveals _configuration_ about the current item

### Label

- Displays static text
- User *cannot* copy

The labels need to be legible at every font size

### Network Activity Indicator

The icon in the status bar

- Spins when activity is on, disappears when it stops
- User interaction is not allowed

### Page Control

Displays how many views are available to view at the bottom of the content body

- Button per view
- Opaque dot displays the current view
- Users _must_ visit in order
- Must be vertically centered

### Progress View

Should display the progress of a task with a _known_ duration

- Should go left to right
- Has two displays: default (main content area) and bar (thinner for use in tool bar)

### Refresh Control

Pull-to-swipe interaction available in table views

- Similar to activity indicator
- Displays an optional title

### Segmented Control

Linear set of buttons available to trigger different views

Remember that…

- Should be easy to tap
- Size of each segment is consistent (**equal widths!**)

### Slider

Range of values from left to right, add custom images to the ends

- Horizontal control with a thumb (circular control)
- Fill track with minimal value at the left

### Stepper

Increase and decrease by a *csontant* amount.

- Doesn’t display changed value
- Supports custom images
- When making large changes, **don’t use a stepper**
- Make it obvious what it affects

### Switch

Off/on

- Binary state
- Only for use in table views

### Buttons

- Use verb/verb phrase for the button
- Titleized title
- Be succinct

### Text Field

Single line of user input

- Customize *only* if it helps the user understand how to use it
- Clear at right end
- Hint in the text field *if* it helps understanding


## Temporary Views

### Alert

Used to notify for _infrequent and important_ information

- Has a title
- One or more buttons
- Avoid unnecessary alerts
- Tell the user what they can do

### Action Sheet

Emerges from bottom of screen and shows available actions for content

- Provides alternate ways to complete a task
- Confirmation before taking a dangerous task
- iPhone should include an easy way to cancel


# Icon and Image Design

Necessary:

- App Icon
- App Icon for App Store
- Launch image

## App Icon

- Simple
- Abstract interpretation of app’s main idea
- Good on various backgrounds
- No transparency

Various sizes are required:

App icon:

- 120x120
- 60x60

App store icon:

- 1024x1024
- 512x512

Spotlight/Settings icon:

For use when the app name is brought up in search or in the Settings app

Spotlight:

- 80x80
- 40x40

Settings

- 58x58
- 29x29

## Launch Image

Must be provided to show a quick and responsive user experience

iPhone 5:

- 640x1136

Other iPhones

- 640x960
- 320x480

