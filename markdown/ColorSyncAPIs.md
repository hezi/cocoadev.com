Can anyone direct me to information regarding using General/ColorSync in cocoa?
Specifically, what I want to do is convert color values with ICC profiles. For example, taking an RGB, CMYK (or LAB value, for that matter) and converting it to some device dependent RGB or CMYK value.
----
We're trying to migrate these General/MailingListMode questions over to the forums (see Quick Links at the top of the page). But that said, General/NSColor will actually do this for you, along with a usually overlooked class called General/NSColorSpace. Check those out for more info.