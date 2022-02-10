# Using Unreal And OpenCV Together
Simple Recount of the Steps Necessary to Use Unreal Engine and OpenCV Together
(Written by someone who spent a lot of time figuring out solutions)

The only reason I am even making this simple repo recounting all of this is that 
while most of my struggles with this can be seen within the Unreal Engine 
Discord, more people are asking me to explain how I solved this problem (after seeing
me talking about it with various people in the Discord), and having this page to 
redirect them to, will be much faster than me repeatidly explaining it. It also
will help those who may be having these issues and are not within the Unreal Discord Community.

# Step 1: Identify the Problem at Hand
When using Unreal Engine either in it's entirety, or as a wrapper for easy code portability, and you want to use OpenCV 
for image analysis or for AR features or camera integration into C++, you need to be aware of one key issue: OpenCV uses vectors.

While this seems innocuous enough, the fact that openCV uses std::vector as it's data representation method is what causes
many crashes that you will experience as soon as you compile your program. It is most notable, but not limited to, the 
"findContours" function that OpenCV provides. This function uses a supplied kernel to extract the contours from an image
and recombine them to make a skeletal structure of the image for further analysis. This is important in things like image
recognition or image extraction.

Unreal Engine's problem is that it contains it's own version of vectors, and does not account for any use of the std library, 
as it supplies alternaties to nearly all of the libraries included within std. So, when it goes to clean up the vectors that
OpenCV makes, it does so carelessly, and immediately goes out of bounds, causing a crash. 

# Step 2 The solution(s)

After talking with various people from the Unreal Engine Discord, I came up with 3 main (C++) ways to fix the problem, and one
language specific way (Python) to remedy the issue as well, at the cost of speed and portability to iOS.

**Solution 1**: 
  This first solution relies on the openCV implementation being given it's own thread to work within.
  If openCV is running on it's own thread (by tying it to an actor or widget component and then instantiating that on a separate thread)
  you can call sleep() or pause() just before unreal goes to clean the thread up. This allows for the indefinite suspension of that thread
  and consequently a delay of the cleanup until the app closes, in which case, the crash can be handled as a correct exit.
  
  I do not recommend this method, although it is easily the quickest to implement, because it causes poor memory usage, since you are leaving 
  a thread and it's entire context within the program for the entirety of it's execution time, which has effects similar to a memory leak if
  the program runs long enough, but can be, like I said before, implemented quickly, and is suitable for a modular program, where each module
  only runs for a small period of time before closing, subsiquently crashing, and handling the crash within Unreal as a proper exit.
  
**Solution 2**:
  The second solution is to do the same thing (giving OpenCV it's own thread), but to kill the thread forcefully before unreal can clean it.
  I found it was easy to do in linux with systemV calls, but never tried to implement a Windows version. However, I'm willing to bet that 
  there is likely a blueprint implementation of threading in the unreal marketplace that probably has a kill() method included with it.
  
  This method is better than the first, but as it stands in my experience, is highly platform dependent, until someone comes up with a cross
  platform method for killing threads, or a blueprint implementation to do the same thing. However, it excels in the sense that it doesn't leave
  threads dangling throughout executon, and while each thread definitely crashes (it's a forced kill), this can be easily ignored since you 
  called the forced kill.
  
**Solution 3**:
  The final way I came up with, which is the most correct way to do it, is to wrap openCV in another library that can handle the cleanup and then 
  allow unreal engine to clean that secondary library up. This can be implemented from scratch, or you could include a module unrelated to the 
  project for the sole purpose of properly cleaning up the vectors before getting cleaned itself by Unreal Engine. This is the only implementation
  that I would deem as 'correct', but can be annoying to implement since now you added yet another layer to your project to manage and keep track of,
  which can cause instability if managed badly, an of course can increase the final executable size if a large and robust package is used to clean the 
  vectors.
  
**Solution 4**:
  The final solution is not what I consider a 'solution' to the problem, but is a suitable workaround. There is a library for Unreal
  that enables the addition of a python VM to the project. This VM has full communication between python scripts and blueprints, which in
  turn can communicate with C++, forming a bridge between Unreal Engine and OpenCV Python. 
  
  This has two main downsides however, the first being that the library used has been unsupported for some time, but has been maintained by
  individuals for a little while now. It supports up to Unreal engine 4.27 through community patching, but is unlikely to make the leap to 
  Unreal Engine 5 when it is released. 
  
  The second problem it has is speed. In my testing, the image recognition took I'd say somewhere in the range of 1.5-3 times longer to process
  since it was going through 2 languages to communicate with Unreal Engine, and since it was being run through an interpreted language rather than
  a compiled one, on an anemic VM running inside of your program. This however, opens the door for other Python Libraries to be used, and may be
  the route you want to take, although, do note that compiling for iOS is not supported (although it seems it will do it), and will not work when the app
  is run. I suspect that support for iOS could be easily added since Python recently made it's debut on iOs itself.
  
  
# Conclusion
While there is no one way that I can say is best for each situation, there is only one I can say is the 'correct' way to do it, which is Solution 3.
However, each solution has its applications and if used correctly can circumvent the large conflict between these two libraries. 

I have also included the links to the binaries for the Unreal Engine Python VM implementations, both official and community patched for your 
use if you decide to go down that route. 

I hope this guide saves you some time and headache, as I know that finding information on this topic is somewhat scarce, and annoying.


# Links
Unreal Engine Python < 2.5 
  https://github.com/20tab/UnrealEnginePython
Unreal Engine Python > 2.5 
  https://github.com/giwig/UnrealEnginePython/releases/tag/20210901
