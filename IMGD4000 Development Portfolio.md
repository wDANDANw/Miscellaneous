
## Abstract
During Spring 2020, D term, I had enrolled in the class IMGD 4000 - Technical Game Development II. We had successfully craft out a playable fighting game with 2 iterations, and this documentation is aimed to retrospect my involvement in the project for future references.

## My Role In The Project
The class hosted during the 2020 pendemic. We had our first trial to put IMGD 4500 Artistic Game Development II & IMGD 4000 Technical Game Development II together. So luckily, we got a chance to work with some artists to come up with a game under simulated studio-like environment. 

I was one of the Gameplay Developer in the development team. More specifically, my job was to 1) Implement Enemy AI, 2) Import Character Assets in [Animations + Meshes], 3) Implement basic character behaviors. [Attack, Jump, etc.].

## Challenges Faced And Our Approches
Since this is my very first trial with Unreal Engine, I did face a lot of challenges. I will try my best to write all of them down here to help future references.

1. Pawn and Character
  At the very beginning, we learned that every object in the scene is an Actor, and Pawn is like a controllable object. But what I didn't know at that time was that there is a child class of Pawn called "Character", which is designed for controllable characters. It has some wrapped in stuffs like movement, which can help your development greatly.

2. How to bind assets to C++ Characters
  Almost in every tutorial you can find, they bind meshes with the character blueprint. But how to initialize a character with pre-set meshes and animations? The answer is to use something called "Constructor helper". Here is some fragements of code you can use:
  In header: 
  ```c++
  #define CHARACTER_SKEL_MESH TEXT(...) [The path to the asset] 
  #define CHARACTER_ANIM_BP TEXT(...) [The path to the asset] 
  ```
  In cpp:  
  ```c++
  static ConstructorHelpers::FObjectFinder<USkeletalMesh> MeshContainer(CHARACTER_SKEL_MESH);
	if (MeshContainer.Succeeded()) {
		GetMesh()->SetSkeletalMesh(MeshContainer.Object);
		GetMesh()->SetRelativeLocation(FVector(0, 0, -87));
	}
  
	static ConstructorHelpers::FObjectFinder<UClass> AnimationBlueprint(CHARACTER_ANIM_BP);
	if (AnimationBlueprint.Object != NULL) GetMesh()->AnimClass = AnimationBlueprint.Object;
  ```
You can get the path to the asset by right click the asset in the content browser in Unreal -> copy reference.

**Notice:** For animation blueprint, for project stability, please cast it to a UClass. You can achieve this by add a \_C at the end of the reference path. For example, 

From : 
```c++
TEXT("AnimBlueprint'/Game/Characters/.../CharacterAnimationBP.CharacterAnimationBP'"))
```
To : 
```c++
TEXT("AnimBlueprint'/Game/Characters/.../CharacterAnimationBP.CharacterAnimationBP_C'"))
```

3. Expose C++ to Blueprint and Character Movement
  You can expose your C++ functions to Blueprint by adding decoraters. Mainly there are three types of decoraters:
  ```c++
  UFUNCTION(BlueprintCallable), UFUNCTION(BlueprintNativeEvent), UFUNCTION(BlueprintImplementable)
  ```
  The first one acts like a function node in blueprint. The second one triggers an event in blueprint. The third one is a little bit tricky, it is defined in C++ and the detailed implementation will be in blueprint.
  You can find more info here: https://www.youtube.com/watch?v=STZxs-NYl7w

  The exposure part is simple, but a weird bug I encountered was that the AI called functions way differently than I expected. 
  I have a movement function called "MoveForward(float amount)", and it is designed to help the character to move forward and backward, depending on the input. However, while it worked pretty well for the character I controlled (player0), the AI just stcuked there moving nowhere while I did put it in for the AI. Luckily by a chance, I found actually the function is being called, and the AI is moving, but with a really really slow rate. My final solution to this problem was to adjust max acceleration of the character (not max speed but the acceleration). So whenever you encounter any movement issues like characters are not moving, please try adjust your max speed and max acceleration first (under movement component of a character. It is a preset component for character).

4. Base Character Class
  As we have multiple characters in the game, it is better to have a base character class for all the characters. It helps us a lot to implement some features that needs dynamic casting. For example, if you have four different character classes for four different fighters, then you need to hardcode a condition checker whenever you want to call a universal function from C++ class as you need absolute casting (from Character to FighterCharacter, for example). Our approach was to establish a base character class and make a few child classes of it. So whenever you want to access any member functions of these classes, you can cast it to the base class.
  
5. How to Implement A Fighter AI
  In Unreal, AI is supported pretty good. We made an AI based on possibility and a decision tree. In unreal, you can craft a decision tree by using "BehaviorTree". For how to craft a decision tree, search "AI Unreal" in youtube and there is already infinite tutorials over there. For decision tree with possibility, you need to craft a random number generator and calculate possibility each run. You can finish this by using blackboard selector (within task to parse out calculated possibility) + blackboard variable (to pass the number around) + blackboard decorator (real time checker for branch).

6. Animation State Machine
  We accomplished our animation with animation state machine blueprint. There are two types of animations: 1) looping animation and 2) one time animation. For example, moving animation is a looping animation since you want to play it when you are moving, while punching is an one time animation since you don't want to it punch all the time while you are not punching. We had a problem with the one time animation, because you cannot simply change the state by checking "isAttacking" is true or false. It turns out that you can achieve this by something called time remaining ratio in animation state machine. Basically you can use isAttacking == true to enter the state, and Time Remaining Ratio <= 0.05 to exit the state. By this way you can avoid loop callings and exit correctly.

## Architectural Diagram For The Project
Check IMGD4000 Final Project.pdf

## Version Control Choice
We chose to use perforce, since it is said that perforce can support binary files (blueprints, level files, etc. in unreal are binary files) better than github. However, we really had a hard time with perforce. Here are some challenges and potential fixes to perforce.

1. Flow
At the beginning sometimes the files are not automatically synced, and we don't know why. After follows to a specific workflow, the sync seemed to be more stable. 
  For files you want to change / modify: Checkout -> Modify -> Save and Compile -> Close Unreal -> Submit Change List
  Especially for binary files, remember to save and compile, and close unreal. It seems that sometimes if you don't close unreal, perforce cannot reconize the latest change and submit it. We had a lot of issues to levels due to this.
  
2. Revert
If you want to discard changes, please use revert. https://www.perforce.com/manuals/v17.1/cmdref/Content/CmdRef/p4_revert.html
This can help you 1) uncheckout files 2) cancel mark for add 3) discard changes 4) fix weird sync bugs

3. Only snyc necessary files
You do not need to sync your binary files, workspace config etc. every time you changed them. What you want to sync are all the changes that are necessary for others to run. For example, contents, assets, c++ classes, etc. What you do not want to sync are those only work for you. For example, build files, workspace config, etc.

## Lessons Learned
Unreal is a really powerful engine with extremly unbelivable run time efficiency. However, I have to say, I did spend a lot of time trying to find one or few lines of code for specific functionality all the time since the documentation is not that good and few people post related questions on the internet (unlike for cs we have stackoverflow). It is expected that you need to spend some time to go over this beginning phase to get to know the beauty of Unreal. But don't forget, your teacher is your best friend whenever you encounter situations like this. Whenever you have problems, ask people with realted experience first. They can save you tons of time.

Also, it is a really good practice to work with art team. This helps us to get familiarize with game studio enviroments before going there. Remember to keep in touch with your team to share your thoughts on the game.

You can certainly create a game individually, but you can create a same game with way less time and better quality with a team. Discuss and chat regularly, then you together can make impressive games!
