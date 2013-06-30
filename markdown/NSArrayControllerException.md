Hi.  I'm getting this stack trace and I'm wondering if someone can help me debug it. I have this type of binding

    
General/NSTableView(value with keyPath name) ->
    General/NSArrayController(contentArray with key path 'shows', the Object Class is Show) ->
        General/NSObjectController.content->General/MyController


General/MyController has a "General/NSMutableArray* shows" attribute and each object in the array is of type Show.
Show has a "General/NSString* name" attribute.

I only get this exception when more than one row exists in the table.  Any ideas?

    
#0	0x90a594c0 in objc_msgSend
#1	0x0038a4c0 in ??
#2	0x9350d91b in -General/[NSValueBinder _adjustObject:mode:observedController:observedKeyPath:context:editableState:adjustState:]
#3	0x93537287 in -General/[NSValueBinder updateTableColumnDataCell:forDisplayAtIndex:]
#4	0x935379ef in -General/[NSTextValueBinder updateTableColumnDataCell:forDisplayAtIndex:]
#5	0x935371d6 in -[_NSBindingAdaptor tableColumn:willDisplayCell:row:]
#6	0x933ffa9d in -General/[NSTableView _sendDelegateWillDisplayCell:forColumn:row:]
#7	0x933e7065 in -General/[NSTableView _drawContentsAtRow:column:clipRect:]
#8	0x933e602b in -General/[NSTableView drawRow:clipRect:]
#9	0x933e37ce in -General/[NSTableView drawRowIndexes:clipRect:]
#10	0x933e26a4 in -General/[NSTableView drawRect:]
#11	0x9330e3b1 in -General/[NSView _drawRect:clip:]
#12	0x9330d40b in -General/[NSView _recursiveDisplayAllDirtyWithLockFocus:visRect:]
#13	0x9331f36f in _recursiveDisplayInRect2
#14	0x9083fb30 in General/CFArrayApplyFunction
#15	0x9330d613 in -General/[NSView _recursiveDisplayAllDirtyWithLockFocus:visRect:]
#16	0x9331f36f in _recursiveDisplayInRect2
#17	0x9083fb30 in General/CFArrayApplyFunction
#18	0x9330d613 in -General/[NSView _recursiveDisplayAllDirtyWithLockFocus:visRect:]
#19	0x9330c473 in -General/[NSView _recursiveDisplayRectIfNeededIgnoringOpacity:isVisibleRect:rectIsVisibleRectForView:topView:]
#20	0x9330d041 in -General/[NSView _recursiveDisplayRectIfNeededIgnoringOpacity:isVisibleRect:rectIsVisibleRectForView:topView:]
#21	0x9330d041 in -General/[NSView _recursiveDisplayRectIfNeededIgnoringOpacity:isVisibleRect:rectIsVisibleRectForView:topView:]
#22	0x9330bb78 in -General/[NSThemeFrame _recursiveDisplayRectIfNeededIgnoringOpacity:isVisibleRect:rectIsVisibleRectForView:topView:]
#23	0x9330b362 in -General/[NSView _displayRectIgnoringOpacity:isVisibleRect:rectIsVisibleRectForView:]
#24	0x9330ac8e in -General/[NSView displayIfNeeded]
#25	0x9330aa32 in -General/[NSWindow displayIfNeeded]
#26	0x9335ad6c in _handleWindowNeedsDisplay
#27	0x9082ed6e in __CFRunLoopDoObservers
#28	0x9082de10 in General/CFRunLoopRunSpecific
#29	0x9082dace in General/CFRunLoopRunInMode
#30	0x92de98d8 in General/RunCurrentEventLoopInMode
#31	0x92de8f19 in General/ReceiveNextEventCommon
#32	0x92de8e39 in General/BlockUntilNextEventMatchingListInMode
#33	0x9328f465 in _DPSNextEvent
#34	0x9328f056 in -General/[NSApplication nextEventMatchingMask:untilDate:inMode:dequeue:]
#35	0x93288ddb in -General/[NSApplication run]
#36	0x9327cd2f in General/NSApplicationMain
#37	0x0000dffc in main at main.m:13