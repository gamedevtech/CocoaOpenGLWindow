
#Part 1, Creating a window
The first thing we need is a __NSAutoreleasePool__, objects declared within it's scope will be automatically released  when the pool is "drained". 
```
NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
```
Next, we create a shared app instance. This will initialize the global variable *NSApp* with the application instance. 
```
[NSApplication sharedApplication]; 
```
Then we create the actual __NSWindow__ object.
```
NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask; 

NSRect windowRect = NSMakeRect(100, 100, 400, 400); // x, y, w, h

NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
					styleMask:windowStyle 
					backing:NSBackingStoreBuffered 
					defer:NO]; 

[window autorelease]; 
```
After the window was created a window controller needs to be created as well
```
NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
[windowController autorelease]; 
```
Finally we show the window and run the event loop, the pool declared above must be drained at this point
```
[window orderFrontRegardless]; 
[NSApp run]; 
[pool drain]
```
###Sample Code (Objective c)
```
#import <Cocoa/Cocoa.h> 
 
int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect windowRect = NSMakeRect(100, 100, 400, 400); 
 
	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 
 
	// TODO: Create app delegate to handle system events.
	// TODO: Create menus (especially Quit!) 
 
	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
 
	[pool drain]; 
 
	return (0); 
}
```
Compile with
```
gcc Cocoa.m -o OSXWindow -framework Cocoa
```
#Part 2, Adding a menu
Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar and can't be brought to the front unless the presentation option is changed
```
[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
```
Next, we need to create the menu bar. You don't need to give the first item in the menubar a name (it will get the application's name automatically)
```
id menubar = [[NSMenu new] autorelease];
id appMenuItem = [[NSMenuItem new] autorelease];
[menubar addItem:appMenuItem];
[NSApp setMainMenu:menubar];
```
Then we add the quit item to the menu. Fortunately the action is simple since terminate: is already implemented in NSApplication and the NSApplication is always in the responder chain.
```
id appMenu = [[NSMenu new] autorelease];
id appName = [[NSProcessInfo processInfo] processName];
id quitTitle = [@"Quit " stringByAppendingString:appName];
id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
	action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
[appMenu addItem:quitMenuItem];
[appMenuItem setSubmenu:appMenu];
```
###Sample Code
```
// gcc Cocoa.m -o OSXWindow -framework Cocoa  
 
#import <Cocoa/Cocoa.h>
 
int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect windowRect = NSMakeRect(100, 100, 400, 400); 
 
	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 

	// Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar 
	// and can't be brought to the front unless the presentation option is changed
	[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
	
	// Next, we need to create the menu bar. You don't need to give the first item in the menubar a name 
	// (it will get the application's name automatically)
	id menubar = [[NSMenu new] autorelease];
	id appMenuItem = [[NSMenuItem new] autorelease];
	[menubar addItem:appMenuItem];
	[NSApp setMainMenu:menubar];

	// Then we add the quit item to the menu. Fortunately the action is simple since terminate: is 
	// already implemented in NSApplication and the NSApplication is always in the responder chain.
	id appMenu = [[NSMenu new] autorelease];
	id appName = [[NSProcessInfo processInfo] processName];
	id quitTitle = [@"Quit " stringByAppendingString:appName];
	id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
		action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
	[appMenu addItem:quitMenuItem];
	[appMenuItem setSubmenu:appMenu];

	// TODO: Create app delegate to handle system events.

	// Set app title
	[window setTitle:appName];

	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
 
	[pool drain]; 
 
	return (0); 
}
```
#Part 3, Events
Unlike in X11 or Win32 Cocoa does not give us explicit control over the event loop. Instead we must create a delegate, and respond to certain events. Lets define a *View* class, this will be both our event delegate and render area.
```
@class View;

@interface View : NSView <NSWindowDelegate> {
@public
	bool running;
}    
@end
```
Now lets start the implementation with the "standard" _initWithFrame_ function.
```
- (id) initWithFrame: (NSRect) frame {
	self = [super initWithFrame:frame];
	running = true;
	// TODO: Init OpenGL
	NSLog(@"initialize");
	
	return self;
}
```
In order to respond to mouse and keyboard events we must be the first responder
```
- (BOOL)acceptsFirstResponder {
	return YES;
}
```
We will respond to the following mouse events __mouseMoved__, __mouseDragged__, __scrollWheel__, __mouseDown__, __mouseUp__, __rightMouseDown__, __rightMouseUp__, __otherMouseDown__, __otherMouseUp__, __mouseEntered__ and __mouseExited__.
```
- (void)mouseMoved:(NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
}

- (void) mouseDragged: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
}

- (void)scrollWheel: (NSEvent*) event  {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse wheel at: %lf, %lf. Delta: %lf", point.x, point.y, [event deltaY] * 10.0);
}

- (void) mouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse down: %lf, %lf", point.x, point.y);
}

- (void) mouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse up: %lf, %lf", point.x, point.y);
}

- (void) rightMouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse down: %lf, %lf", point.x, point.y);
}

- (void) rightMouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse up: %lf, %lf", point.x, point.y);
}

- (void)otherMouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse down: %lf, %lf", point.x, point.y);
}

- (void)otherMouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse up: %lf, %lf", point.x, point.y);
}

- (void) mouseEntered: (NSEvent*)event {
	NSLog(@"Mouse entered");
}

- (void) mouseExited: (NSEvent*)event {
	NSLog(@"Mouse left");
}
```
We will process keyboard events with __keyDown__ and __keyUp__
```
- (void) keyDown: (NSEvent*) event {
	if ([event isARepeat] == NO) {
		NSLog(@"Key down: %d", [event keyCode]);
	}
}

- (void) keyUp: (NSEvent*) event {
	NSLog(@"Key up: %d", [event keyCode]);
}
```
Lets also respond to the window being resized
```
// Resize
- (void)windowDidResize:(NSNotification*)notification {
	NSSize size = [ [ _window contentView ] frame ].size;
	NSLog(@"Window resize: %lf, %lf", size.width, size.height);
}
```
We will handle cleanup in the __windowWillClose__ function, we also force the app to terminate. This will cause the app to close when teh red X is pressed. Also, implement a stock __dealloc__ function
```
-(void)windowWillClose:(NSNotification *)notification {
	if (running) {
		running = false;

		NSLog(@"Cleanup");
	}

	[NSApp terminate:self];
}

- (void) dealloc {   
	[super dealloc];
}
```
Inside the main function, we create a View, and set it's delegate
```
View* view = [[[View alloc] initWithFrame:windowRect] autorelease];
[window setAcceptsMouseMovedEvents:YES];
[window setContentView:view];
[window setDelegate:view];
```
### Sample Code
```
// gcc Cocoa.m -o OSXWindow -framework Cocoa
 
#import <Cocoa/Cocoa.h>

@class View;

@interface View : NSView <NSWindowDelegate> {
@public
	bool running;
}    
@end

@implementation View
// Initialize
- (id) initWithFrame: (NSRect) frame {
	self = [super initWithFrame:frame];
	running = true;
	// TODO: Init OpenGL

	NSLog(@"initialize");

	return self;
}

// Tell the window to accept input events
- (BOOL)acceptsFirstResponder {
	return YES;
}

- (void)mouseMoved:(NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
}

- (void) mouseDragged: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
}

- (void)scrollWheel: (NSEvent*) event  {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse wheel at: %lf, %lf. Delta: %lf", point.x, point.y, [event deltaY] * 10.0);
}

- (void) mouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse down: %lf, %lf", point.x, point.y);
}

- (void) mouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse up: %lf, %lf", point.x, point.y);
}

- (void) rightMouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse down: %lf, %lf", point.x, point.y);
}

- (void) rightMouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse up: %lf, %lf", point.x, point.y);
}

- (void)otherMouseDown: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse down: %lf, %lf", point.x, point.y);
}

- (void)otherMouseUp: (NSEvent*) event {
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse up: %lf, %lf", point.x, point.y);
}

- (void) mouseEntered: (NSEvent*)event {
	NSLog(@"Mouse entered");
}

- (void) mouseExited: (NSEvent*)event {
	NSLog(@"Mouse left");
}

- (void) keyDown: (NSEvent*) event {
	if ([event isARepeat] == NO) {
		NSLog(@"Key down: %d", [event keyCode]);
	}
}

- (void) keyUp: (NSEvent*) event {
	NSLog(@"Key up: %d", [event keyCode]);
}

// Resize
- (void)windowDidResize:(NSNotification*)notification {
	NSSize size = [ [ _window contentView ] frame ].size;
	NSLog(@"Window resize: %lf, %lf", size.width, size.height);
}

// Terminate window when the red X is pressed
-(void)windowWillClose:(NSNotification *)notification {
	if (running) {
		running = false;

		NSLog(@"Cleanup");
	}

	[NSApp terminate:self];
}

// Cleanup
- (void) dealloc {   
	[super dealloc];
}
@end

int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask | NSMiniaturizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect screenRect = [[NSScreen mainScreen] frame];
	NSRect viewRect = NSMakeRect(0, 0, 800, 600); 
	NSRect windowRect = NSMakeRect(NSMidX(screenRect) - NSMidX(viewRect),
								 NSMidY(screenRect) - NSMidY(viewRect),
								 viewRect.size.width, 
								 viewRect.size.height);

	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 

	// Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar 
	// and can't be brought to the front unless the presentation option is changed
	[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
	
	// Next, we need to create the menu bar. You don't need to give the first item in the menubar a name 
	// (it will get the application's name automatically)
	id menubar = [[NSMenu new] autorelease];
	id appMenuItem = [[NSMenuItem new] autorelease];
	[menubar addItem:appMenuItem];
	[NSApp setMainMenu:menubar];

	// Then we add the quit item to the menu. Fortunately the action is simple since terminate: is 
	// already implemented in NSApplication and the NSApplication is always in the responder chain.
	id appMenu = [[NSMenu new] autorelease];
	id appName = [[NSProcessInfo processInfo] processName];
	id quitTitle = [@"Quit " stringByAppendingString:appName];
	id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
		action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
	[appMenu addItem:quitMenuItem];
	[appMenuItem setSubmenu:appMenu];

	// Create app delegate to handle system events
	View* view = [[[View alloc] initWithFrame:windowRect] autorelease];
	[window setAcceptsMouseMovedEvents:YES];
	[window setContentView:view];
	[window setDelegate:view];

	// Set app title
	[window setTitle:appName];

	// Add fullscreen button
	[window setCollectionBehavior: NSWindowCollectionBehaviorFullScreenPrimary];

	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
	
	[pool drain]; 
 
	return (0); 
}
```
#Part 3, Update
We're going to use the quartz display link to update. First, the __CVDisplayLink__ header needs to be included
```
#import <QuartzCore/CVDisplayLink.h>
```
Next, forward declare the function that will re-schedule update
```
static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef, const CVTimeStamp*, const CVTimeStamp*, CVOptionFlags, CVOptionFlags*, void*);
```
Inside our view class, include a __CVDisplayLinkRef__ variable.
```
@interface View : NSView <NSWindowDelegate> {
@public
	CVDisplayLinkRef displayLink;
	bool running;
}    
@end
```
Set the link up in init display
```
CVDisplayLinkCreateWithActiveCGDisplays(&displayLink);
CVDisplayLinkSetOutputCallback(displayLink, &GlobalDisplayLinkCallback, self);
CVDisplayLinkStart(displayLink);
```
Then we implement the __getFrameForTime__ function
```
- (CVReturn) getFrameForTime:(const CVTimeStamp*)outputTime {
	NSLog(@"Update");
	if (false) { // Update loop returns false
		[NSApp terminate:self];
	}
	return kCVReturnSuccess;
}
```
Inside the *windowWillClose* function, the display link needs to be killed
```
CVDisplayLinkStop(displayLink);
CVDisplayLinkRelease(displayLink);
```
Finally the actual _GlobalDisplayLinkCallback_ function that was declared earlyer
```
static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef displayLink, const CVTimeStamp* now, const CVTimeStamp* outputTime, CVOptionFlags flagsIn, CVOptionFlags* flagsOut, void* displayLinkContext) {
	CVReturn result = [(View*)displayLinkContext getFrameForTime:outputTime];
	return result;
}
```
### Sample Code
```
// gcc Cocoa.m -o OSXWindow -framework Cocoa -framework Quartz
 
#import <Cocoa/Cocoa.h>
#import <QuartzCore/CVDisplayLink.h>

@class View;
static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef, const CVTimeStamp*, const CVTimeStamp*, CVOptionFlags, CVOptionFlags*, void*);

@interface View : NSView <NSWindowDelegate> {
@public
	CVDisplayLinkRef displayLink;
	bool running;
}    
@end

@implementation View
// Initialize
- (id) initWithFrame: (NSRect) frame {
	self = [super initWithFrame:frame];
	running = true;
	// TODO: Init OpenGL

	NSLog(@"initialize");

	CVDisplayLinkCreateWithActiveCGDisplays(&displayLink);
	CVDisplayLinkSetOutputCallback(displayLink, &GlobalDisplayLinkCallback, self);
	CVDisplayLinkStart(displayLink);
	
	return self;
}

// Update
- (CVReturn) getFrameForTime:(const CVTimeStamp*)outputTime {
	NSLog(@"Update");
	if (false) { // Update loop returns false
		[NSApp terminate:self];
	}
	return kCVReturnSuccess;
}

// Terminate window when the red X is pressed
-(void)windowWillClose:(NSNotification *)notification {
	if (running) {
		running = false;
		CVDisplayLinkStop(displayLink);
		CVDisplayLinkRelease(displayLink);

		NSLog(@"Cleanup");
	}

	[NSApp terminate:self];
}

// Cleanup
- (void) dealloc {   
	[super dealloc];
}
@end

static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef displayLink, const CVTimeStamp* now, const CVTimeStamp* outputTime, CVOptionFlags flagsIn, CVOptionFlags* flagsOut, void* displayLinkContext) {
	CVReturn result = [(View*)displayLinkContext getFrameForTime:outputTime];
	return result;
}

int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask | NSMiniaturizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect screenRect = [[NSScreen mainScreen] frame];
	NSRect viewRect = NSMakeRect(0, 0, 800, 600); 
	NSRect windowRect = NSMakeRect(NSMidX(screenRect) - NSMidX(viewRect),
								 NSMidY(screenRect) - NSMidY(viewRect),
								 viewRect.size.width, 
								 viewRect.size.height);

	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 

	// Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar 
	// and can't be brought to the front unless the presentation option is changed
	[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
	
	// Next, we need to create the menu bar. You don't need to give the first item in the menubar a name 
	// (it will get the application's name automatically)
	id menubar = [[NSMenu new] autorelease];
	id appMenuItem = [[NSMenuItem new] autorelease];
	[menubar addItem:appMenuItem];
	[NSApp setMainMenu:menubar];

	// Then we add the quit item to the menu. Fortunately the action is simple since terminate: is 
	// already implemented in NSApplication and the NSApplication is always in the responder chain.
	id appMenu = [[NSMenu new] autorelease];
	id appName = [[NSProcessInfo processInfo] processName];
	id quitTitle = [@"Quit " stringByAppendingString:appName];
	id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
		action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
	[appMenu addItem:quitMenuItem];
	[appMenuItem setSubmenu:appMenu];

	// Create app delegate to handle system events
	View* view = [[[View alloc] initWithFrame:windowRect] autorelease];
	[window setAcceptsMouseMovedEvents:YES];
	[window setContentView:view];
	[window setDelegate:view];

	// Set app title
	[window setTitle:appName];

	// Add fullscreen button
	[window setCollectionBehavior: NSWindowCollectionBehaviorFullScreenPrimary];

	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
	
	[pool drain]; 
 
	return (0); 
}
```
#Part 4, Getting an OpenGL Context
First lets include two additional headers for OpenGL
```
#import <OpenGL/OpenGL.h>
#include <OpenGL/gl.h>
```
Next, change the __View__ class to extend __NSOpenGLView__ instead of __NSView__. We will also add two variables to the class, one that is the current Window Rect, and a lock so we can update and draw safely.
```
@interface View : NSOpenGLView <NSWindowDelegate> {
@public
	CVDisplayLinkRef displayLink;
	bool running;
	NSRect windowRect;
	NSRecursiveLock* appLock;
}    
@end
```
Insize *initWithFrame* we will try to make a new pixel format. This format is needed to create the new OpenGL window as _super:initWithFrame_ will now take a pixel format. We also remove the code to create the *CVDisplayLink*
```
// No multisampling
int samples = 0;

// Keep multisampling attributes at the start of the attribute lists since code below assumes they are array elements 0 through 4.
NSOpenGLPixelFormatAttribute windowedAttrs[] = 
{
	NSOpenGLPFAMultisample,
	NSOpenGLPFASampleBuffers, samples ? 1 : 0,
	NSOpenGLPFASamples, samples,
	NSOpenGLPFAAccelerated,
	NSOpenGLPFADoubleBuffer,
	NSOpenGLPFAColorSize, 32,
	NSOpenGLPFADepthSize, 24,
	NSOpenGLPFAAlphaSize, 8,
	NSOpenGLPFAOpenGLProfile, NSOpenGLProfileVersionLegacy,
	0
};

// Try to choose a supported pixel format
NSOpenGLPixelFormat* pf = [[NSOpenGLPixelFormat alloc] initWithAttributes:windowedAttrs];

if (!pf) {
	NSLog(@"OpenGL pixel format not supported.");
	return nil;
}

self = [super initWithFrame:frame pixelFormat:[pf autorelease]];
```
Finally, we initialize the lock
```
appLock = [[NSRecursiveLock alloc] init];
```
Next we must implement the __prepareOpenGL__ method inherited from *NSOpenGLView*. First, call the same method of the base class, then we will set the OpenGL context of the view as the current context and turn on VSynch. We also move the code to create a display link into this section of the application, the display link will be synched to our OpenGL refresh rate. Finally, we start the display link.
```
- (void) prepareOpenGL {
	[super prepareOpenGL];
		
	// Make all the OpenGL calls to setup rendering and build the necessary rendering objects
	[[self openGLContext] makeCurrentContext];
	// Synchronize buffer swaps with vertical refresh rate
	GLint swapInt = 1; // Vsynch on!
	[[self openGLContext] setValues:&swapInt forParameter:NSOpenGLCPSwapInterval];
	
	// Create a display link capable of being used with all active displays
	CVDisplayLinkCreateWithActiveCGDisplays(&displayLink);
	
	// Set the renderer output callback function
	CVDisplayLinkSetOutputCallback(displayLink, &GlobalDisplayLinkCallback, self);
	
	CGLContextObj cglContext = (CGLContextObj)[[self openGLContext] CGLContextObj];
	CGLPixelFormatObj cglPixelFormat = (CGLPixelFormatObj)[[self pixelFormat] CGLPixelFormatObj];
	CVDisplayLinkSetCurrentCGDisplayFromOpenGLContext(displayLink, cglContext, cglPixelFormat);
	
	GLint dim[2] = {windowRect.size.width, windowRect.size.height};
	CGLSetParameter(cglContext, kCGLCPSurfaceBackingSize, dim);
	CGLEnable(cglContext, kCGLCESurfaceBackingSize);
	
	[appLock lock];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Initialize");

	NSLog(@"GL version:   %s", glGetString(GL_VERSION));
	NSLog(@"GLSL version: %s", glGetString(GL_SHADING_LANGUAGE_VERSION));

	glClearColor(0.5f, 0.6f, 0.7f, 1.0f);
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);
	glEnable(GL_DEPTH_TEST);

	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];

	// Activate the display link
	CVDisplayLinkStart(displayLink);
}
```
Every method we implemented will need to lock and unlock appLock like the above code sample does. The *getFrameForTime* method will simply clear the frame color for now. Once the update is finished we present the context
```
// Update
- (CVReturn) getFrameForTime:(const CVTimeStamp*)outputTime {
	[appLock lock];
	
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);

	NSLog(@"Update");
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	NSLog(@"Render");

	CGLFlushDrawable((CGLContextObj)[[self openGLContext] CGLContextObj]);
	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 

	if (false) { // Update loop returns false
		[NSApp terminate:self];
	}

	[appLock unlock];
	
	return kCVReturnSuccess;
}
```
The *windowDidResize* function will now keep track of the window size, and set the glViewPort
```
- (void)windowDidResize:(NSNotification*)notification {
	NSSize size = [ [ _window contentView ] frame ].size;
	[appLock lock];
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Window resize: %lf, %lf", size.width, size.height);

	windowRect.size.width = size.width;
	windowRect.size.height = size.height;
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);

	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];
}
```
We introduce two more inherited methods, __resumeDisplayRenderer__ and __haltDisplayRenderer__. Both of these methods are simply going to lock *appLock*, stop the *CVDispayLink* and unlock.
```
[appLock lock];
CVDisplayLinkStop(displayLink);
[appLock unlock];
```
The __windowWillClose__ method must lock and unlock before stopping the display link as well
```
if (running) {
	running = false;

	[appLock lock];
	NSLog(@"Cleanup");

	CVDisplayLinkStop(displayLink);
	CVDisplayLinkRelease(displayLink);

	[appLock unlock];
}
```
And finally, we release the appLock inside of the dealloc function
```
- (void) dealloc {   
	[appLock release];
	[super dealloc];
}
```
###Sample Code
```
// gcc Make.m -o MWindow -framework Cocoa -framework Quartz -framework OpenGL
 
#import <Cocoa/Cocoa.h>
#import <QuartzCore/CVDisplayLink.h>
#import <OpenGL/OpenGL.h>
#include <OpenGL/gl.h>

@class View;
static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef, const CVTimeStamp*, const CVTimeStamp*, CVOptionFlags, CVOptionFlags*, void*);

@interface View : NSOpenGLView <NSWindowDelegate> {
@public
	CVDisplayLinkRef displayLink;
	bool running;
	NSRect windowRect;
	NSRecursiveLock* appLock;
}    
@end

@implementation View
// Initialize
- (id) initWithFrame: (NSRect) frame {
	running = true;
	
	// No multisampling
	int samples = 0;

	// Keep multisampling attributes at the start of the attribute lists since code below assumes they are array elements 0 through 4.
	NSOpenGLPixelFormatAttribute windowedAttrs[] = 
	{
		NSOpenGLPFAMultisample,
		NSOpenGLPFASampleBuffers, samples ? 1 : 0,
		NSOpenGLPFASamples, samples,
		NSOpenGLPFAAccelerated,
		NSOpenGLPFADoubleBuffer,
		NSOpenGLPFAColorSize, 32,
		NSOpenGLPFADepthSize, 24,
		NSOpenGLPFAAlphaSize, 8,
		NSOpenGLPFAOpenGLProfile, NSOpenGLProfileVersionLegacy,
		0
	};

	// Try to choose a supported pixel format
	NSOpenGLPixelFormat* pf = [[NSOpenGLPixelFormat alloc] initWithAttributes:windowedAttrs];

	if (!pf) {
		NSLog(@"OpenGL pixel format not supported.");
		return nil;
	}
	
	self = [super initWithFrame:frame pixelFormat:[pf autorelease]];
	appLock = [[NSRecursiveLock alloc] init];

	return self;
}

- (void) prepareOpenGL {
	[super prepareOpenGL];
		
	// Make all the OpenGL calls to setup rendering and build the necessary rendering objects
	[[self openGLContext] makeCurrentContext];
	// Synchronize buffer swaps with vertical refresh rate
	GLint swapInt = 1; // Vsynch on!
	[[self openGLContext] setValues:&swapInt forParameter:NSOpenGLCPSwapInterval];
	
	// Create a display link capable of being used with all active displays
	CVDisplayLinkCreateWithActiveCGDisplays(&displayLink);
	
	// Set the renderer output callback function
	CVDisplayLinkSetOutputCallback(displayLink, &GlobalDisplayLinkCallback, self);
	
	CGLContextObj cglContext = (CGLContextObj)[[self openGLContext] CGLContextObj];
	CGLPixelFormatObj cglPixelFormat = (CGLPixelFormatObj)[[self pixelFormat] CGLPixelFormatObj];
	CVDisplayLinkSetCurrentCGDisplayFromOpenGLContext(displayLink, cglContext, cglPixelFormat);
	
	GLint dim[2] = {windowRect.size.width, windowRect.size.height};
	CGLSetParameter(cglContext, kCGLCPSurfaceBackingSize, dim);
	CGLEnable(cglContext, kCGLCESurfaceBackingSize);
	
	[appLock lock];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Initialize");

	NSLog(@"GL version:   %s", glGetString(GL_VERSION));
	NSLog(@"GLSL version: %s", glGetString(GL_SHADING_LANGUAGE_VERSION));

	glClearColor(0.5f, 0.6f, 0.7f, 1.0f);
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);
	glEnable(GL_DEPTH_TEST);

	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];

	// Activate the display link
	CVDisplayLinkStart(displayLink);
}

// Tell the window to accept input events
- (BOOL)acceptsFirstResponder {
	return YES;
}

// Update
- (CVReturn) getFrameForTime:(const CVTimeStamp*)outputTime {
	[appLock lock];
	
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);

	NSLog(@"Update");
	// Temp
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
	// EndTemp
	NSLog(@"Render");
	
	CGLFlushDrawable((CGLContextObj)[[self openGLContext] CGLContextObj]);
	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 

	if (false) { // Update loop returns false
		[NSApp terminate:self];
	}

	[appLock unlock];
	
	return kCVReturnSuccess;
}

// Resize
- (void)windowDidResize:(NSNotification*)notification {
	NSSize size = [ [ _window contentView ] frame ].size;
	[appLock lock];
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Window resize: %lf, %lf", size.width, size.height);
	// Temp
	windowRect.size.width = size.width;
	windowRect.size.height = size.height;
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);
	// End temp
	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];
}

- (void)resumeDisplayRenderer  {
	[appLock lock];
	CVDisplayLinkStop(displayLink);
	[appLock unlock]; 
}

- (void)haltDisplayRenderer  {
	[appLock lock];
	CVDisplayLinkStop(displayLink);
	[appLock unlock];
}

// Terminate window when the red X is pressed
-(void)windowWillClose:(NSNotification *)notification {
	if (running) {
		running = false;

		[appLock lock];
		NSLog(@"Cleanup");

		CVDisplayLinkStop(displayLink);
		CVDisplayLinkRelease(displayLink);

		[appLock unlock];
	}

	[NSApp terminate:self];
}

// Cleanup
- (void) dealloc {   
	[appLock release];
	[super dealloc];
}
@end

static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef displayLink, const CVTimeStamp* now, const CVTimeStamp* outputTime, CVOptionFlags flagsIn, CVOptionFlags* flagsOut, void* displayLinkContext) {
	CVReturn result = [(View*)displayLinkContext getFrameForTime:outputTime];
	return result;
}

int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask | NSMiniaturizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect screenRect = [[NSScreen mainScreen] frame];
	NSRect viewRect = NSMakeRect(0, 0, 800, 600); 
	NSRect windowRect = NSMakeRect(NSMidX(screenRect) - NSMidX(viewRect),
								 NSMidY(screenRect) - NSMidY(viewRect),
								 viewRect.size.width, 
								 viewRect.size.height);

	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 

	// Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar 
	// and can't be brought to the front unless the presentation option is changed
	[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
	
	// Next, we need to create the menu bar. You don't need to give the first item in the menubar a name 
	// (it will get the application's name automatically)
	id menubar = [[NSMenu new] autorelease];
	id appMenuItem = [[NSMenuItem new] autorelease];
	[menubar addItem:appMenuItem];
	[NSApp setMainMenu:menubar];

	// Then we add the quit item to the menu. Fortunately the action is simple since terminate: is 
	// already implemented in NSApplication and the NSApplication is always in the responder chain.
	id appMenu = [[NSMenu new] autorelease];
	id appName = [[NSProcessInfo processInfo] processName];
	id quitTitle = [@"Quit " stringByAppendingString:appName];
	id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
		action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
	[appMenu addItem:quitMenuItem];
	[appMenuItem setSubmenu:appMenu];

	// Create app delegate to handle system events
	View* view = [[[View alloc] initWithFrame:windowRect] autorelease];
	view->windowRect = windowRect;
	[window setAcceptsMouseMovedEvents:YES];
	[window setContentView:view];
	[window setDelegate:view];

	// Set app title
	[window setTitle:appName];

	// Add fullscreen button
	[window setCollectionBehavior: NSWindowCollectionBehaviorFullScreenPrimary];

	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
	
	[pool drain]; 
 
	return (0); 
}
```

#Part 5, Modern OpenGL Context
The above is great, but running it on mavericks it gives us an OpenGL 2.1 context with support for GLSL 1.2. While this is good for testing mobile device compatibality; we want to get a modern OpenGL Context (3.0 +).
In order to do this, change the __gl.h__ include to __gl3.h__ 
```
#include <OpenGL/gl3.h>
```
And in the __NSOpenGLPixelFormatAttribute windowedAttrs__ variable change __NSOpenGLPFAOpenGLProfile__ to *NSOpenGLProfileVersion3_2Core*.
```
NSOpenGLPixelFormatAttribute windowedAttrs[] = {
	NSOpenGLPFAMultisample,
	NSOpenGLPFASampleBuffers, samples ? 1 : 0,
	NSOpenGLPFASamples, samples,
	NSOpenGLPFAAccelerated,
	NSOpenGLPFADoubleBuffer,
	NSOpenGLPFAColorSize, 32,
	NSOpenGLPFADepthSize, 24,
	NSOpenGLPFAAlphaSize, 8,
	NSOpenGLPFAOpenGLProfile, NSOpenGLProfileVersion3_2Core,
	0
};
```
That's it. You should now have a 3.2+ context. On My MBA it gives me a 4.2 context. The below "Putting it all together" section will use the legacy context.

#Part 6, Putting it all together
```
// gcc Cocoa.m -o OSXWindow -framework Cocoa -framework Quartz -framework OpenGL
 
#import <Cocoa/Cocoa.h>
#import <QuartzCore/CVDisplayLink.h>
#import <OpenGL/OpenGL.h>
#include <OpenGL/gl.h>

@class View;
static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef, const CVTimeStamp*, const CVTimeStamp*, CVOptionFlags, CVOptionFlags*, void*);

@interface View : NSOpenGLView <NSWindowDelegate> {
@public
	CVDisplayLinkRef displayLink;
	bool running;
	NSRect windowRect;
	NSRecursiveLock* appLock;
}    
@end

@implementation View
// Initialize
- (id) initWithFrame: (NSRect) frame {
	running = true;
	
	// No multisampling
	int samples = 0;

	// Keep multisampling attributes at the start of the attribute lists since code below assumes they are array elements 0 through 4.
	NSOpenGLPixelFormatAttribute windowedAttrs[] = 
	{
		NSOpenGLPFAMultisample,
		NSOpenGLPFASampleBuffers, samples ? 1 : 0,
		NSOpenGLPFASamples, samples,
		NSOpenGLPFAAccelerated,
		NSOpenGLPFADoubleBuffer,
		NSOpenGLPFAColorSize, 32,
		NSOpenGLPFADepthSize, 24,
		NSOpenGLPFAAlphaSize, 8,
		NSOpenGLPFAOpenGLProfile, NSOpenGLProfileVersionLegacy,
		0
	};

	// Try to choose a supported pixel format
	NSOpenGLPixelFormat* pf = [[NSOpenGLPixelFormat alloc] initWithAttributes:windowedAttrs];

	if (!pf) {
		bool valid = false;
		while (!pf && samples > 0) {
			samples /= 2;
			windowedAttrs[2] = samples ? 1 : 0;
			windowedAttrs[4] = samples;
			pf = [[NSOpenGLPixelFormat alloc] initWithAttributes:windowedAttrs];
			if (pf) {
				valid = true;
				break;
			}
		}
		
		if (!valid) {
			NSLog(@"OpenGL pixel format not supported.");
			return nil;
		}
	}
	
	self = [super initWithFrame:frame pixelFormat:[pf autorelease]];
	appLock = [[NSRecursiveLock alloc] init];

	return self;
}

- (void) prepareOpenGL {
	[super prepareOpenGL];
		
	[[self window] setLevel: NSNormalWindowLevel];
	[[self window] makeKeyAndOrderFront: self];

	// Activate the application (i.e. give it focus).
	[NSApp activateIgnoringOtherApps:YES];
	
	// Make all the OpenGL calls to setup rendering and build the necessary rendering objects
	[[self openGLContext] makeCurrentContext];
	// Synchronize buffer swaps with vertical refresh rate
	GLint swapInt = 1; // Vsynch on!
	[[self openGLContext] setValues:&swapInt forParameter:NSOpenGLCPSwapInterval];
	
	// Create a display link capable of being used with all active displays
	CVDisplayLinkCreateWithActiveCGDisplays(&displayLink);
	
	// Set the renderer output callback function
	CVDisplayLinkSetOutputCallback(displayLink, &GlobalDisplayLinkCallback, self);
	
	CGLContextObj cglContext = (CGLContextObj)[[self openGLContext] CGLContextObj];
	CGLPixelFormatObj cglPixelFormat = (CGLPixelFormatObj)[[self pixelFormat] CGLPixelFormatObj];
	CVDisplayLinkSetCurrentCGDisplayFromOpenGLContext(displayLink, cglContext, cglPixelFormat);
	
	GLint dim[2] = {windowRect.size.width, windowRect.size.height};
	CGLSetParameter(cglContext, kCGLCPSurfaceBackingSize, dim);
	CGLEnable(cglContext, kCGLCESurfaceBackingSize);
	
	[appLock lock];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Initialize");

	NSLog(@"GL version:   %s", glGetString(GL_VERSION));
    NSLog(@"GLSL version: %s", glGetString(GL_SHADING_LANGUAGE_VERSION));
	// Temp
	glClearColor(0.5f, 0.6f, 0.7f, 1.0f);
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);
	glEnable(GL_DEPTH_TEST);
	// End temp
	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];

	// Activate the display link
	CVDisplayLinkStart(displayLink);
}

// Tell the window to accept input events
- (BOOL)acceptsFirstResponder {
	return YES;
}

- (void)mouseMoved:(NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void) mouseDragged: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse pos: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void)scrollWheel: (NSEvent*) event  {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Mouse wheel at: %lf, %lf. Delta: %lf", point.x, point.y, [event deltaY]);
	[appLock unlock];
}

- (void) mouseDown: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse down: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void) mouseUp: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Left mouse up: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void) rightMouseDown: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse down: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void) rightMouseUp: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Right mouse up: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void)otherMouseDown: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse down: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void)otherMouseUp: (NSEvent*) event {
	[appLock lock];
	NSPoint point = [self convertPoint:[event locationInWindow] fromView:nil];
	NSLog(@"Middle mouse up: %lf, %lf", point.x, point.y);
	[appLock unlock];
}

- (void) mouseEntered: (NSEvent*)event {
	[appLock lock];
	NSLog(@"Mouse entered");
	[appLock unlock];
}

- (void) mouseExited: (NSEvent*)event {
	[appLock lock];
	NSLog(@"Mouse left");
	[appLock unlock];
}

- (void) keyDown: (NSEvent*) event {
	[appLock lock];
	if ([event isARepeat] == NO) {
		NSLog(@"Key down: %d", [event keyCode]);
	}
	[appLock unlock];
}

- (void) keyUp: (NSEvent*) event {
	[appLock lock];
	NSLog(@"Key up: %d", [event keyCode]);
	[appLock unlock];
}

// Update
- (CVReturn) getFrameForTime:(const CVTimeStamp*)outputTime {
	[appLock lock];
	
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);

	NSLog(@"Update");
	// Temp
	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

	// EndTemp

	CGLFlushDrawable((CGLContextObj)[[self openGLContext] CGLContextObj]);
    CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 

    if (false) { // Update loop returns false
		[NSApp terminate:self];
	}

	[appLock unlock];
	
	return kCVReturnSuccess;
}

// Resize
- (void)windowDidResize:(NSNotification*)notification {
	NSSize size = [ [ _window contentView ] frame ].size;
	[appLock lock];
	[[self openGLContext] makeCurrentContext];
	CGLLockContext((CGLContextObj)[[self openGLContext] CGLContextObj]);
	NSLog(@"Window resize: %lf, %lf", size.width, size.height);
	// Temp
	windowRect.size.width = size.width;
	windowRect.size.height = size.height;
	glViewport(0, 0, windowRect.size.width, windowRect.size.height);
	// End temp
	CGLUnlockContext((CGLContextObj)[[self openGLContext] CGLContextObj]); 
	[appLock unlock];
}

- (void)resumeDisplayRenderer  {
    [appLock lock];
    CVDisplayLinkStop(displayLink);
    [appLock unlock]; 
}

- (void)haltDisplayRenderer  {
    [appLock lock];
    CVDisplayLinkStop(displayLink);
    [appLock unlock];
}

// Terminate window when the red X is pressed
-(void)windowWillClose:(NSNotification *)notification {
	if (running) {
		running = false;

		[appLock lock];
		NSLog(@"Cleanup");

		CVDisplayLinkStop(displayLink);
		CVDisplayLinkRelease(displayLink);

		[appLock unlock];
	}

	[NSApp terminate:self];
}

// Cleanup
- (void) dealloc {  
	[appLock release]; 
	[super dealloc];
}
@end

static CVReturn GlobalDisplayLinkCallback(CVDisplayLinkRef displayLink, const CVTimeStamp* now, const CVTimeStamp* outputTime, CVOptionFlags flagsIn, CVOptionFlags* flagsOut, void* displayLinkContext) {
	CVReturn result = [(View*)displayLinkContext getFrameForTime:outputTime];
	return result;
}

int main(int argc, const char * argv[])  { 
	// Autorelease Pool: 
	// Objects declared in this scope will be automatically 
	// released at the end of it, when the pool is "drained". 
	NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init]; 
 
	// Create a shared app instance. 
	// This will initialize the global variable 
	// 'NSApp' with the application instance. 
	[NSApplication sharedApplication]; 
 
	// Create a window: 
 
	// Style flags 
	NSUInteger windowStyle = NSTitledWindowMask  | NSClosableWindowMask | NSResizableWindowMask | NSMiniaturizableWindowMask; 

	// Window bounds (x, y, width, height) 
	NSRect screenRect = [[NSScreen mainScreen] frame];
	NSRect viewRect = NSMakeRect(0, 0, 800, 600); 
	NSRect windowRect = NSMakeRect(NSMidX(screenRect) - NSMidX(viewRect),
								 NSMidY(screenRect) - NSMidY(viewRect),
								 viewRect.size.width, 
								 viewRect.size.height);

	NSWindow * window = [[NSWindow alloc] initWithContentRect:windowRect 
						styleMask:windowStyle 
						backing:NSBackingStoreBuffered 
						defer:NO]; 
	[window autorelease]; 
 
	// Window controller 
	NSWindowController * windowController = [[NSWindowController alloc] initWithWindow:window]; 
	[windowController autorelease]; 

	// Since Snow Leopard, programs without application bundles and Info.plist files don't get a menubar 
	// and can't be brought to the front unless the presentation option is changed
	[NSApp setActivationPolicy:NSApplicationActivationPolicyRegular];
	
	// Next, we need to create the menu bar. You don't need to give the first item in the menubar a name 
	// (it will get the application's name automatically)
	id menubar = [[NSMenu new] autorelease];
	id appMenuItem = [[NSMenuItem new] autorelease];
	[menubar addItem:appMenuItem];
	[NSApp setMainMenu:menubar];

	// Then we add the quit item to the menu. Fortunately the action is simple since terminate: is 
	// already implemented in NSApplication and the NSApplication is always in the responder chain.
	id appMenu = [[NSMenu new] autorelease];
	id appName = [[NSProcessInfo processInfo] processName];
	id quitTitle = [@"Quit " stringByAppendingString:appName];
	id quitMenuItem = [[[NSMenuItem alloc] initWithTitle:quitTitle
		action:@selector(terminate:) keyEquivalent:@"q"] autorelease];
	[appMenu addItem:quitMenuItem];
	[appMenuItem setSubmenu:appMenu];

	// Create app delegate to handle system events
	View* view = [[[View alloc] initWithFrame:windowRect] autorelease];
	view->windowRect = windowRect;
	[window setAcceptsMouseMovedEvents:YES];
	[window setContentView:view];
	[window setDelegate:view];

	// Set app title
	[window setTitle:appName];

	// Add fullscreen button
	[window setCollectionBehavior: NSWindowCollectionBehaviorFullScreenPrimary];

	// Show window and run event loop 
	[window orderFrontRegardless]; 
	[NSApp run]; 
	
	[pool drain]; 
 
	return (0); 
}
```

#Sources
* https://github.com/gameplay3d/GamePlay/blob/master/gameplay%2Fsrc%2FPlatformMacOSX.mm
* http://www.mightwerk.com/blog/DC261D88-555B-494B-B563-DAC1271776FB/index.html
* https://developer.apple.com/library/mac/qa/qa1385/_index.html
* https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/EventOverview/HandlingKeyEvents/HandlingKeyEvents.html
* http://fredandrandall.com/blog/2011/09/08/how-to-make-your-app-open-in-full-screen-on-lion/
* https://casperbhansen.wordpress.com/2010/08/15/dev-tip-nibless-development/
* http://stackoverflow.com/questions/8233141/completely-close-an-os-x-application-with-window-close-button
* http://stackoverflow.com/questions/626898/how-do-i-create-delegates-in-objective-c
* http://en.wikibooks.org/wiki/Programming_Mac_OS_X_with_Cocoa_for_Beginners/Wikidraw's_view_class