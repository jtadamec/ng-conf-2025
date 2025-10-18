Wednesday - Performance Workshop


Presented by [Push Based](https://github.com/push-based)

# Render Pipeline

## Steps 
- Javascript 
- Recalculate Styles
  - costly 
- Layout
  - Flexbox can cause many recusions of drawing 
- Paint 
  - color
  - border-color 
  - border-radius 
    - does not cause a layout change because it does not change a location
- Composite 
  - Slices the website into different "pictures" based on the layout into layers  
  - Intended to not have to redraw the entire website if something minor changes in one area of the site
    - Opacity, translate/transform, border radius
  - Significantly faster
  - Runs on GPU instead of CPU  
  - Example: changing the color of a div would cause the paint layer to run. But changing the opacity of a layer and transitioning to a different layer would run only in the composite step 

Javascript and Recalculate steps always run, but the others can be limited
<br>
[CSS Triggers explanations](https://www.lmame-geek.com/css-triggers/)

# Core Web Vitals 

Largest Contentful Paint
- Loading performance
- Measures time until page's main content has loaded
- Is the user consuming it?

Interaction to Next Paint
- Reactivity
- How long does it take until a tooltip shows up after hovering over a box? 

Cumulate Layout Shift
- Visual Stability 
- Trust in the experience 

## LCP
A "good" LCP is up to 2.5 seconds. After that, the user will wonder if there is an issue with their internet, problem with the website, etc. Anything above 4 seconds will cause the user to refer to the website as "slow" and produce a negative reaction.

### LCP Candidates
- Imgs, block level elements (h1, h2, p, etc)
- Things with a width and height 
- between the load event, the browser render pipeline, etc until the first frame is presented the webpage starts to take shape
  - When the browser is idle after the frame is presented, it looks for the last biggest candidate in the viewport 

Server Side Rendering can be used to avoid lengthy LCP cycles

Larger viewports can make the LCP longer since there is more to display

### LCP Breakdown 

**Phases**
- Time to first byte
  - Disconnected from the real content - internet connection, etc
- Resource Load Delay 
  - Example: having to fetch JSON which itself has the urls for images that need to be loaded
  - Another Example: execute some javascript that triggers requests on a subpage 
- Resource Load Time
- Element Render Delay
  - Scripts, css, etc that do things to the browser that delay making an image, etc. viewable 

Looking at the breakdown to know that the largest delay in the website load time is not (necessarily) the image size, but if scripts are triggering scripts that trigger loads, etc... 

Largest bottlenecks tend to be in step #2 and step #4 

## Interaction to Next Paint 
*Replaced FID (First Input Delay) in March 24*

< 200ms is good, > 500ms is slow 

- Combination of input delay, processing time, and presentation delay (Render Pipeline triggers) up until the frame is painted and presented to the user 
- If operations are long running, make things asynchronous to be non blocking and paint a loading indicator in the place of where content will appear 
  - Tell the user *something* as early as possible so that they don't think the browser is hung, the internet is slow, etc. 

**TBT**: Total Blocking Time 
- Anything more than 50ms becomes a "long task" 
  - As described by the [Rail Model](https://web.dev/articles/rail)
- How to turn blocking tasks into non-blocking 
  - Don't do the work ("deleting code")
    - aka make sure that the stuff you're doing actually needs to be done
  - Defer work 
  - Prioritize the more important work first

## Cumulative Layout Shift 

- A spatial calculation. Things that are shifted and not visible to the user are not penalized in the calculation
- < .1 is good, > .25 is slow 

To prevent CLS, reserve space for things that will be loading in. 
- Telerik skeleton component, for example 
  - Can also improve INP by loading a skeleton as a long running request runs in the background
- Loading spinner 
  - When in doubt spin it out 
- ng utilities for lazy loading images, prioritize

Custom fonts are a contributor to CLS and are hard to optimize
- [Custom tools are available](https://meowni.ca/font-style-matcher/) to try and match a standard font size's letter spacing, kerning, etc. to your custom one to reduce shift once the custom font does finish loading

## Other Web Vitals
- Time To First Byte
- First Contentful Paint
  - First impressions can matter
- Total Blocking Time 
  - TBT can be used to "derive" a feeling about INP because it aggregates all the things that can cause bad INP
- Time To Interactive 
  - At the beginning of the loading, how long does it take until the user can click a button
  - No longer "core", similarly to First Input Delay 

Measure these things and more in the **Performance** tab in Chrome

**CrUX**: [Chrome User Experience](https://developer.chrome.com/docs/crux/dashboard) metrics. Can measure incremental progress when improving a website over time. 
- RunVision can be installed locally to measure internal sites that are not available for external analysis
  - But browsers can also natively calculate this fairly well now (refer to performance tab notes)

## Tooling Documentation
[CPU Profiling in the Browser](https://github.com/push-based/cpu-prof/blob/main/packages/cpu-prof/docs/cpu-profiling.md)

# Concurrency Model and Event Loop

**queueMicrotask** is native functionality for scheduling code within events 
- Microtasks are always completed before other event handling or rendering 
  - Browser can guarantee that the environment does not change inbetween async code running

Timer operations are **Macrotasks** that get queued at the end of the event loop to run when next available 

# Mastering the Single Thread

~16.66ms allowed between frames to get to 60fps

**RAIL**: Response Animation Idle Load

Use scheduling to keep the page loading quickly:
- Time
- IdleCallback
- postTask
- MessageChannel 

Long tasks can block the main thread and create long INP times. To improve it
- Do less work
  - Not always feasible if you truly need to do everything that is being called  
- Distribute it / chunk it
  - Split the work into multiple threads 
- Prioritise it

**RxAngular**: Implements a Concurrent Mode ("borrowed" from React where things are done in 16ms chunks by default)
- Frame budget aims for 60fps, but can be configured down for busy apps 
- Splitting into chunks allows the user to click inbetween those 16ms chunks (or whatever the config size is) to queue their actions immediately instead of having to wait for a long running task to complete
- Note: this appears to be written by the people who are presenting so take its necessity with a grain of salt... 

```import RxLet```

Improper use of the directive can cause content to shift around 

Q&A: 
- **RxLet vs defer** 
  - RxLet is used to chunk the template, while a standard defer is used to tell the browser that it can be loaded later as a less important task
- **General rules of thumb for scheduling**
  - When the profiler shows that an interaction is taking too long (for example)
    - Measure first before guessing that something is an issue
  - Update a long running task to split up the work so that loading spinners are used to provide feedback on long running items
- **RxLet decreases time of task? Or just prevent flagging as long running?**
  - Answer: it only chunks something up into smaller tasks that aren't as long running, it doesn't reduce the total execution time. 
  - Chunking it does remove it from the main thread so that it can be ran in the future
- **Difference between rxJs and Angular, and this**
  - Nothing like this is present for template splitting in the default library
  - This doesn't affect state management like a signalr store
  - ngExtensions also has some state helpers
    - even the framework libraries have libraries, yo dawg
- **Can combine with defer block?**
  - Yes, rxLet can be deferred which then chunks up items 
- **Compare to virtual scrolling**
  - In VS, they are in a list but not yet rendered. This is for things that should be on the viewport already 

Try out the library in [their code sample](https://github.com/push-based/perf-playground/tree/main)

There is also an rxFor example to split a for loop item into multiple tasks

# Signals 
A wrapper around a value that can notifiy consumers when the value changes.

Keywords:
- Notify
- Consumers
- Any value
  - Map, array, string, object, etc....

## Producers and Consumers

Under the hood creates a reactivity graph between producers and consumers. 

```const count  = signal(0)``` is the producer.

```WritableSignals``` add on set, update, and asReadonly functions to the basic Signal.

```effect```s are the consumers of signals, as well as angular templates.
- The ng template reexecutes when it sees a signal change

The ```computed``` signal is both a producer and a consumer because it watches a signal to see a value change and then itself produces a new value. 

Producers and Consumers are ReactiveNode

Effects are "live consumers" because they are listening immediately when they are created. Computeds, by contrast, are not live consumers because set cannot be called on them. If a computed is not called by a component or template then it is not live and listening. 

## Push and Pull Behavior 
When producers change they __push__ a notification that a value has changed.

When a live consumer gets a notification of a change, they are scheduled on a microtask queue to go __pull__ the latest value.
- Computeds are not calculated immediately therefore - it has to first go fetch the value as a pull
- Slightly confusing, see slide 35 of into to signals for the graph 

Signals do not require a subscribe (key difference from observables - they therefore do not need to be destroyed). There is an implicit subscription when a signal is called within an effect that angular manages for you. 

### Demo

[Basic Custom Signal Example](https://stackblitz.com/edit/stackblitz-starters-t9xjrl8p)

- Create a signal
- Create an effect
- The effect calls the signal to get the value 
- Angular keeps track of the number of active consumers in a global variable

# Angular Change Detection
Angular is a component tree framework that uses ZoneJs to detect changes

```bootstrapApplication``` produces the ```AppRef``` instance that contains these tools. 

Angular change updates are for looking what actually changed and modifying as small a part of the DOM as possible. 

```tick()``` is used to traverse the component tree to see if anything has changed.

## ZoneJs
ZoneJs can be thought of as a wrapper around the native API
- addEventListener
- alert
- XMLHttpRequest

etc...
Angular Apps + SetInterval can therefore cause a lot of issues since the set interval is itself wrapped.

There are problems with the approach of API patching, since scrolling and other high-frequency events will cause way too many events. 
- Some of these can be disabled in the polyfill file 
- There is ```ngZone.runOutsideAngular``` to run outside the scope created by ZoneJs
  - Similar functionality exists in signalrs with runUntracked
  - The original functions are also available with double underscores in the global window (since ZoneJs still needs to actually invoke them)
    - don't do that btw

## On Push

ChangeDetectionStrategy.OnPush can be used as an alternative to tell the parent that there are changes instead of running the detection on an interval. This is _explicit_ change detection
- Note that this is basically what signals do for us in new angular 
- Bottom up dirty marking, then top down rendering 
- When some components are marked for automatic changes and another one is marked with OnPush, it is not automatically re-rendered when other components are detected to have changes
  - Real smart dude running this demo recommends always using OnPush instead of relying on angular to do it for you
    - This feels like in my Blazor app calling an event that says I changed something to then trigger a redraw

## markForCheck
Component marks itself as needing to be checked for changes, and the parent components are also marked
- When combined with onPush, creates a broken state if the parent does not notify that it should be re-rendered

async pipe change detection uses markForCheck under the hood 

random note that was brought up: install AngularDevTools as a chrome extension for profiling... we're gonna need some permissions for that though.

Mark for check is recommended over calling "detect changes" on the CDR (change detector ref)

ALL OF THIS (according to this smart dude) were magic tricks that angular used before signals existed

## Signals
A feature exists to mark components for checking if they read a signal that changes.

New change detection mode was introduced alongside signals. OnPush + HasChildViewsToRefresh, calls RefreshView to look for signals that have changed. In global mode, this still traverses children in the "old" way
- Hilariously, the example given was that this super old component didn't know about this feature when it was created THREE years ago. 
- This supports migrating to using signals gradually, since child components that use the "old" method will still be updated 
- ExpressionChangedAfterItHasBeenCheckedError GOES AWAY with signals
  - Signals caps the looping at 100, old method will just loop forever

**Note**: There is an ESLint rule to make your signals and computeds readonly to help enforce proper usage

Signals + OnPush changes produces targeted, local changes. If you are fully using signals, you also don't need ZoneJs since ticks don't need to be scheduled on a busyloop 
- ZoneJs used to be needed to mark something for check and then call tick later
- ```NotificationSource``` enum contains all the sources that can call tick 

In version 21, angular will be zoneless by default. 
- The presenter literally said "then angular can be normal again" with no magic, like the others
- This removes the magic "patchBrowser" call happens before *anything* can happen inside the app
  - Because it has to happen upfront in all applications, this will improve performance of *every* angular app that used to use it
- You will have to add back zones if you or your libraries are depending on it 
