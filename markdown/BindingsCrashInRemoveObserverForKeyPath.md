I'm seeing an odd and frustrating bindings related crash, with the following stack:

    
#0	0x90812125 in CFRetain
#1	0x92894a9d in _NSKeyValueObservationInfoCreateByRemoving
#2	0x92894888 in -[NSObject(NSKeyValueObserverRegistration) _removeObserver:forProperty:]
#3	0x92894800 in -[NSObject(NSKeyValueObserverRegistration) removeObserver:forKeyPath:]
#4	0x92894d10 in -[NSKeyValueObservationForwarder observeValueForKeyPath:ofObject:change:context:]
#5	0x927df344 in -[NSObject(NSKeyValueObserverNotification) didChangeValueForKey:]


Google shows up various references to similar crashes, but none of the threads seems to lead to a fix. I've got a test case that captures it nicely every time. See below for source code link.

The context of the crash is as follows.

I have an array of objects (call them items), and an NSMatrix subclass that they are displayed in. 

The NSMatrix has an instance variable which is the NSArrayController for the array, and observes     arrangedObjects on the array controller. The NSMatrix iterates over the arranged objects initially, or when they change (i.e. when     -observeValueForKeyPath:ofObject:change:context: is called with the correct args) setting the representedObject of each cell to the underlying item, and also setting the text value for the cell appropriately.

I also have an info panel, which displays info for the currently selected item in the NSMatrix. The infopanel fields are actually bound to the windowcontroller, with the path     matrix.selectedItem.name. The windowcontroller has an appropriate accessor for the matrix. When the selected item in the NSMatrix subclass changes, the matrix calls     -willChangeValueForKey: and     -didChangeValueForKey: for     selectedItem.

This all works very nicely. Hooray for bindings! I can arrow key around the matrix (with a bit more custom code), with the info window updating effortlessly, and if items are added to the array, the matrix picks them up automatically.

The problem comes when I start deleting items. I catch delete key presses in the matrix's keyDown: method, retrieve the represented item from the cell, remove it from the array controller, receive the arrangedObject change notification, and update all the cells with new titles and represented objects. Then I programmatically select an appropriate cell, and call the      -willChangeValueForKey: and     -didChangeValueForKey: for     selectedItem, to ensure the info panel is notified.

This is where the crash happens. A fuller stack is:

    
#0	0x90812125 in CFRetain
#1	0x92894a9d in _NSKeyValueObservationInfoCreateByRemoving
#2	0x92894888 in -[NSObject(NSKeyValueObserverRegistration) _removeObserver:forProperty:]
#3	0x92894800 in -[NSObject(NSKeyValueObserverRegistration) removeObserver:forKeyPath:]
#4	0x92894d10 in -[NSKeyValueObservationForwarder observeValueForKeyPath:ofObject:change:context:]
#5	0x927df344 in -[NSObject(NSKeyValueObserverNotification) didChangeValueForKey:]
#6	0x00043ee2 in -[BCDMatrix selectCellAtRow:column:] at BCDMatrixView.m:118
#7	0x00044190 in -[BCDMatrix deleteSelectedBCDObject] at BCDMatrixView.m:190
#8	0x00044286 in -[BCDMatrix keyDown:] at BCDMatrixView.m:219


(The key is actually "selectedBCDObject", but I've used "selectedItem" above for simplicity).

Interestingly, if I delete the last item, it doesn't crash. This remains true, for the new last item as well. Deleting any item except for the last one always crashes it.

Commenting out the     xxxxChangeValueForKey: calls stops the crash as well. I used     poseAsClass: to dump out the args to the removeObserver calls, and can see that its a     NSKeyValueObservationForwarder observing of the item object, with the final part of the key paths for the info panel. The first of these triggers the crash.

    
2007-04-24 03:42:20.576 Sentry[21458] removing observer <NSKeyValueObservationForwarder: 0x3d3bd0> for keyPath fileName from <MMIRecording: 0x366d70>
2007-04-24 03:42:20.576 Sentry[21458] removing observer <NSKeyValueObservationForwarder: 0x3d3bd0> for property <NSKeyValueUnnestedProperty: Container class: MMIRecording, Key path: fileName, isa for autonotifying: (null), Other trigger keys: (null)> from <MMIRecording: 0x366d70>


However, I'm taking care to retain the item throughout deletion, so the underlying item itself is still there. The "item" object always returns NO from     automaticallyNotifiesObserversForKey:, so there's nothing funky going on with autogenerated classes.

CFZombieLevel didn't reveal very much about what CFRetain actually had problems with.

Sample code can be found at http://www.mildmanneredindustries.com/downloads/BindingsCrashTest.zip 

BCDMatrixView is where all the action happens. Just arrow around in the matrix, and try hitting delete on the last item, and then on a non-last item.

----

You are changing the arrangedObjectsArray in observeValueForKeyPath: and one of the items you are changing is being referred too from "selectedBCDObject". Try changing your code to this (at~BCDMatrixView.m:310)

    
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    NSLog(@"observeValueForKeyPath:%@ ofObject:%@ change:%@ context:%p", keyPath, object, change, context);

    if(object != mObjectArrayController || [keyPath isEqualToString:@"arrangedObjects"] == NO)
    {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
    else
    {
        NSEnumerator *objectEnumerator = mObjectArrayController arrangedObjects] objectEnumerator];
        [[BCDObject *theObject = nil;
        int i = 0;
        int selectedRow = [self selectedRow];
        int selectedColumn = [self selectedColumn];

        while(theObject = [objectEnumerator nextObject]){
            NSLog(@"rejigging cell for %d", i);
            int row = i / [self numberOfColumns];
            int column = i - (row * [self numberOfColumns]);
            NSCell *textCell = [self cellAtRow:row column:column];
            if (selectedRow == row && selectedColumn == column) {
              [self willChangeValueForKey:@"selectedBCDObject"];
            }
            [textCell setObjectValue:[theObject name]];
            [textCell setRepresentedObject:theObject];
            if (selectedRow == row && selectedColumn == column) {
              [self didChangeValueForKey:@"selectedBCDObject"];    
            }
            i++;
        }
        
        while(i < ([self numberOfRows] * [self numberOfColumns]))
        {
            NSLog(@"Removing cell %d", i);            
            int row = i / [self numberOfColumns];
            int column = i - (row * [self numberOfColumns]); 
             [self putCell:[[NSActionCell alloc] init] atRow:row column:column];            
            i++;
        }
        
    }
}


That will update things correctly I think.

DaveMacLachlan
