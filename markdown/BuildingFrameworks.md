I'm building frameworks, and I'm deploying to /Library/Frameworks/MyFramework.framework and PB likes to set a -w perm on the framework preventing my from cleaning and building without kicking back to a terminal and either sudoing or chmod u+w on MyFramework... which is annoying to say the least.  I can also sudo pbxbuild and what not, but is there anything I can do to say out of the terminal and have PB either not set those perms or have it unset those perms on rebuild and clean?  Thanks!  Anybody?

----

Do you really want to give PB free reign over system level directories? It's probably not a bad idea to manually move things to avoid shotting yourself in the foot.

----

Yes.  I really want PB to clean and rebuild when I say clean and rebuild -- the only thing it can possibly delete is its own target build.  Anyhow, I found the answer to my own question.  DEPLOYMENT_POSTPROCESSING should be set to NO.  If someone else wanted to know the answer to this question, but doesn't feel comfortable with open ended perms, have it install into $(HOME)/Library/Frameworks/ instead of /Library/Frameworks/ (which is probably where you'd want to deploy final builds).

----

Different question, same topic.

How does one build a framework that only uses Foundation?

I have some code I want to put it in a framework. I noticed it only uses Foundation. I started it a Cocoa application. So I want to remove unnecessary references to AppKit.

I first
* changed all #import <Cocoa/Cocoa.h> generated by Xcode to  #import <Foundation/Foundation.h>
* removed references to Cocoa and AppKit framework from "External Frameworks And Libraries > Other Frameworks"
I then compiled but the linker complained about missing symbols (all Foundation symbols).

so I moved Foundation.Framework to "Targets > MyFramework > Frameworks" and it compiled fine.
But when I do a clean all, and rebuild, it spends a fair while precompiling AppKit. And there is some 35MB of shared cache taken up by appkit in the build directory. Growl. 

Question is, is there anything wrong with doing this? Will Foundation.framework be included in the built product?

----

FYI, The Apple developer info for building frameworks can be found at http://developer.apple.com/documentation/MacOSX/Conceptual/BPFrameworks/index.html - DerekBolli
