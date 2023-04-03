# -CLIENT-Wizard
Game Engine

Game Development has seen rapid growth the past few decades. I
have been playing video games since I was a child, since forever I was cap-
tivated by their complexity and how they were made, to no surprise as I
grew up I began creating my own games. In this paper I will explore the
inner workings of the 2D Games a lot of people enjoy, from the low level
systems required, to the necessary code written to create a 2D game with an
TCP/IP CLIENT/SERVER networking architecture. I’m calling this game
“The Wizard”.

1.1 Simple DirectMedia Layer 2
SDL2 is an open-source library that gives low level access to audio,
keyboard, mouse and graphics hardware.In this specific use case it allows the
use and conversion of 2D images into Textures that can be manipulated with
a variety of functions, it allows the merging of multiple textures into a single
texture. SDL also provides multi-threading support, which is necessary in
creating an project such as this.

Foundational Systems
2.1 Windows
In order to draw and poll events an “Window” is required.
“SDL Window*” is an SDL structure that holds the attributes of the win-
dow, we give this structure our properties using “SDL CreateWindow”, as
paramenters we must specify Window Size(X and Y), the Window name and
a number of “flags” are also accepted.

void LWindow::handleEvent(SDL_Event& e)
{
	//In case an event was detected
	if (e.type == SDL_WINDOWEVENT && e.window.windowID == mWindowID)
	{
		//Caption update flag
		bool updateCaption = false;
		switch (e.window.event)
		{
			//Window is shown on screen
		case SDL_WINDOWEVENT_SHOWN:
			mShown = true;
			break;

			//Window is not shown on screen
		case SDL_WINDOWEVENT_HIDDEN:
			mShown = false;
			break;

			//Change window size
		case SDL_WINDOWEVENT_SIZE_CHANGED:
			mWidth = e.window.data1;
			mHeight = e.window.data2;
			SDL_RenderPresent(mRenderer);
			break;

			//Repaint on expose
		case SDL_WINDOWEVENT_EXPOSED:
			SDL_RenderPresent(mRenderer);
			break;

			//In case mouse enters the window frame 
		case SDL_WINDOWEVENT_ENTER:
			mMouseFocus = true;
			updateCaption = true;
			break;
            ...
			//Hide the window on close
		case SDL_WINDOWEVENT_CLOSE:
			SDL_HideWindow(mWindow);
			break;
		}

	}
}

2.2 Renderer
In order to draw images onto an window we first need an renderer.
“SDL Renderer” is the structure SDL provides to make this possible, we give
it attributes with “SDL CreateRenderer” which takes an “SDL Window*” as
one of it’s parameters, binding the Renderer and Window together.

bool LWindow::init()
{
	//Create window
	if (mFullScreen)
	{
		mWindow = SDL_CreateWindow("The Wizard", SDL_WINDOWPOS_UNDEFINED, SDL_WINDOWPOS_UNDEFINED, mWidth, mHeight, SDL_WINDOW_SHOWN | SDL_WINDOW_FULLSCREEN);
	}
	else
	{
		mWindow = SDL_CreateWindow("The Wizard", SDL_WINDOWPOS_UNDEFINED, SDL_WINDOWPOS_UNDEFINED, mWidth, mHeight, SDL_WINDOW_SHOWN);
	}
	
	if (mWindow != NULL)
	{
		mMouseFocus = true;
		mKeyboardFocus = true;

		//Create renderer for window
		mRenderer = SDL_CreateRenderer(mWindow, -1, SDL_RENDERER_ACCELERATED);
		if (mRenderer == NULL)
		{
			std::cout << std::endl << "Renderer could not be created! SDL Error:\n" << SDL_GetError();
			SDL_DestroyWindow(mWindow);
			mWindow = NULL;
		}
		else
		{
			//Initialize renderer color
			SDL_SetRenderDrawColor(mRenderer, 0xFF, 0xFF, 0xFF, 0xFF);

			//Grab window identifier
			mWindowID = SDL_GetWindowID(mWindow);

			//Flag as opened
			mShown = true;

		}
	}
	else
	{
		std::cout << std::endl << "Window could not be created! SDL Error: \n" << SDL_GetError();
	}

	return mWindow != NULL && mRenderer != NULL;
}


Textures
Textures are what we are going to display on the Window using our
Renderer. “SDL Texture” is the structure that allows us to do this.First step
is to load the desired image into memory, second we are going to convert the
image into an surface, using “SDL Surface” we can specify an RGB Map
that allows us to convert specific pixel colors to transparent pixels, which
will not be rendered on the Window.The next step is converting this surface
to a texture, giving it an pixel size and allowing us to make further changes
to the pixel structure.


bool LTexture::loadFromFile(std::string path, SDL_Renderer* gRenderer)
{
	//Get rid of preexisting texture
	free();

	//The final texture
	SDL_Texture* newTexture = NULL;

	//Load image at specified path
	SDL_Surface* loadedSurface = IMG_Load(path.c_str());
	if (loadedSurface == NULL)
	{
		cout << "Unable to load image SDL Error:\n" << IMG_GetError();
	}
	else
	{
		//Color key image
		SDL_SetColorKey(loadedSurface, SDL_TRUE, SDL_MapRGB(loadedSurface->format, 255, 25, 255));

		//Create texture from surface pixels
		newTexture = SDL_CreateTextureFromSurface(gRenderer, loadedSurface);
		if (newTexture == NULL)
		{
			cout << "Unable to create texture SDL Error:\n" << SDL_GetError();
		}
		else
		{
			//Get image dimensions
			mWidth = loadedSurface->w;
			mHeight = loadedSurface->h;
			collisionBox.w = mWidth;
			collisionBox.h = mHeight;
		}

		//Get rid of old loaded surface
		SDL_FreeSurface(loadedSurface);
	}

	//Return success
	mTexture = newTexture;
	return mTexture != NULL;
}

Timers
Timers are a requirement for a Game Engine, SDL provides the
“SDL Timer” library for this. This library allows us to have multiple mil-
lisecond precise timers running simultaneously independent of the program
Up Time and the System Clock.
Most programs run on a single CPU Thread, in game development
that is not possible without major limitations.For the creation of this gamem
ultiple threads are necessary, these threads run code in parallel, sharing
memory.Separate threads for sending and receiving data are necessary, as for
the game there is a mainloop thread and an physics Thread, this allows us to
separate the game Frames per Second (how many times a second all textures
are rendered) from the physics speed.

Multi Threading
Most programs run on a single CPU Thread, in game development
that is not possible without major limitations.For the creation of this game
multiple threads are necessary, these threads run code in parallel, sharing
memory.Separate threads for sending and receiving data are necessary, as for
the game there is a mainloop thread and an physics Thread, this allows us to
separate the game Frames per Second (how many times a second all textures
are rendered) from the physics speed.


Handling player input
Object Oriented Programming is a very useful tool in creating an
Game Engine. In our game ”The Wizard” we want the player to be able to
move freely on the screen,to do that we need to take mouse and keyboard
inputs from the player.
Using “SDL Event” we are able to poll each action the player
takes. When a event is triggered a function runs that reflects the desired
effect.The keys ”W”,”A”,”S”,”D” are used to control the Pawn in an 2D
manner.“W” will bring the Pawn up on the screen and “S” will bring the
Pawn down.While “A” and “D” are used to move the pawn on the Y axis.
But limitations are necessary, we need the Pawn to remain within Window
borders, as such we verify that the input from the player will place the Pawn
within Window limits.We also need to control the speed at which the Pawn
can move, in order to do that the Physics Thread limits how many times a
second the pawn can move.

You can also limit the speed by capping the frame rate to a fixed
number, although doing that can cause problems in cases where the computer
can not reach the frame rate set, slowing the physics to an unintended level.
The “LPawn” class has a variety of attributes such as the X and Y positions,
data about the state of the Pawn such as death state, heading position,
nickname, unique ID, health, if the Pawn is moving, the Pawn collision data
and more.

void LPawn::handleEvent(SDL_Event& e,SDL_Point mapSize)
{
moving = false;
if (currentKeyStates[SDL_SCANCODE_W] && mCollider.y > 1)
{
mCollider.y -= DEFAULT_VEL;
moving = true;
}
if (currentKeyStates[SDL_SCANCODE_S] && mCollider.y < mapSize.x
&& (mCollider.y + 0 < mapSize.y-100))
{
mCollider.y += DEFAULT_VEL;
moving = true;
}
if (currentKeyStates[SDL_SCANCODE_A] && mCollider.x > 1)
{
mCollider.x -= DEFAULT_VEL;
charDir = true;
moving = true;
}
if (currentKeyStates[SDL_SCANCODE_D] && mCollider.y < mapSize.y
&& (mCollider.x + 0 < mapSize.x-100))
{
mCollider.x += DEFAULT_VEL;
charDir = false;
moving = true;
}
p_collisionRect.x = mCollider.x + xCollisionOffset;
p_collisionRect.y = mCollider.y + yCollisionOffset;
}

Firing projectiles
In this game the player is able to fire projectiles from the Pawn posi-
tions by using the Left Mouse Button at a specified position.The LProjectile
objects are declared inside the LPawn class, such that every Pawn has an
fixed ammount of LProjectile objects available. The moment a Left Mouse
Click event is detected the position of the mouse cursor within the window
is passed to a function that creates the projectile.A timer is used in order to
limit the amount of projectiles that can be fired within a certain period of
time.
Creating projectile and determining it’s 


void LPawn::spawnProjectile(int x, int y, int ang, int dx, int dy,
float projSpeed)
{
for (unsigned int i = 0; i < MAX_PROJECTILES; i++)
{
if (gProjectile[i].getSlotFree())
{
gProjectile[i].setDestX(dx);
gProjectile[i].setDestY(dy);
gProjectile[i].setSlotFree(false);
gProjectile[i].setPosX(x);
gProjectile[i].setPosY(y);
gProjectile[i].setAngle(90 + (atan2(dy - y, dx - x) * 180
/ 3.14f));
gProjectile[i].DISTANCE = sqrt(pow(dx - x, 2) + pow(dy -
y, 2));
mVelX = (projSpeed / gProjectile[i].DISTANCE) * (dx - x);
mVelY = (projSpeed / gProjectile[i].DISTANCE) * (dy - y);
gProjectile[i].setVelX(mVelX);
gProjectile[i].setVelY(mVelY);
break;
}
}
}


Animations
Animations are a necessity in modern games, in order to create an
animation we need multiple images and a way to combine them in an concise
object that can be called on at any time. The “LAnim” class is created
in order to make this possible. Each “LAnim” object can hold multiple
textures, we use a function to load all images into memory and convert them
to textures by giving the function the path to the images and the ammount
of frames we want the animation to have, in this case the ammount of
frames constitutes the ammount of images loaded. We can procedurally
load these images from a folder by giving them the exact same naming
structure only changing the numbers (image 001,image 002...image xyz).In
“The Wizard” animations are used for the controllable character when:
doing nothing,moving,firing an projectile. The projectile fired from the
player is in itself a animation.
This engine has two ways to create animations, the first is where
each texture is a separate file, loaded sequentially and converted into a ani-
mation. The second is where the animation consists of a single file, I call this
a cropped animation. In this situation you need to know give the function
the size of the animation, so that the engine knows how much to crop for
each texture and then convert it into a animation.


void LAnim::renderStaticAnim(SDL_Renderer * gRenderer,bool renderCollisiionBox, int colW, int colH,int colX, int colY)
{
	//RENDER SEQ ANIM

	if (!p_isCrop)
	{
		for (unsigned int i = 0; i < MAX_VARIABLE_AMMOUNT; i++)
		{
			for (unsigned int j = 0; j < MAX_VARIABLE_AMMOUNT; j++)
			{
				if (seqInUse[i][j] && isInverseSeq[i][j] == false)
				{	
					animTexture[currentTickClient[i][j]].render(gRenderer, seqPosX[i][j], seqPosY[i][j], NULL, animAngle[i][j], NULL, SDL_FLIP_NONE, renderCollisiionBox, colW, colH,colX,colY);

					if (animTimer[i][j].getTicks() > 500 / tickCount)
					{
						currentTickClient[i][j]++;
						animTimer[i][j].reset();
					}
					if (currentTickClient[i][j] >= tickCount - 1)
					{
						currentTickClient[i][j] = 0;
						isInverseSeq[i][j] = true;
					}
				}
				else if (isInverseSeq[i][j] && p_renderInverse)
				{
					animTexture[tickCount - currentTickClient[i][j]].render(gRenderer, seqPosX[i][j], seqPosY[i][j], NULL, animAngle[i][j], NULL, SDL_FLIP_NONE, renderCollisiionBox, colW, colH,colX,colY);

					if (animTimer[i][j].getTicks() > 500 / tickCount)
					{
						currentTickClient[i][j]++;
						animTimer[i][j].reset();
					}
					if (currentTickClient[i][j] >= tickCount - 1)
					{
						currentTickClient[i][j] = 0;
						isInverseSeq[i][j] = false;
						seqInUse[i][j] = false;
					}
				}
			}
		}
	}
	else
	{
		//RENDER CROPPED ANIMS
		for (unsigned int i = 0; i < MAX_VARIABLE_AMMOUNT; i++)
		{
			if (cropInUse[i] && !isInverse[i])
			{
				animTexture[0].render(gRenderer, cropPosX[i], cropPosY[i], &animRect[currentTickClient[0][i]], 0, 0, SDL_FLIP_NONE,renderCollisiionBox, colW, colH,colX,colY);

				if (animTimer[0][i].getTicks() > tickTime / tickCount)
				{
					currentTickClient[0][i]++;
					animTimer[0][i].reset();
				}
				if (currentTickClient[0][i] >= tickCount - 1)
				{
					animTimer[0][i].reset();
					currentTickClient[0][i] = 0;

					if (p_renderInverse)
					{
						isInverse[i] = true;
					}
					else
					{
						cropInUse[i] = false;
					}
				}
			}
			else if (isInverse[i] && p_renderInverse)
			{
				animTexture[0].render(gRenderer, cropPosX[i], cropPosY[i], &animRect[tickCount - currentTickClient[0][i]], 0, 0, SDL_FLIP_NONE,renderCollisiionBox, colW, colH,colX,colY);

				if (animTimer[0][i].getTicks() > tickTime / tickCount)
				{
					currentTickClient[0][i]++;
					animTimer[0][i].reset();
				}
				if (currentTickClient[0][i] >= tickCount - 1)
				{
					animTimer[0][i].reset();
					currentTickClient[0][i] = 0;
					cropInUse[i] = false;
					isInverse[i] = false;
				}
			}
		}
	}
}
Game development is a vast and complex industry, every year new and
more advanced systems are created to build games. Even thought most of
the systems created by me were done before, it doesn’t necessarily mean that
they must never be redone again. By programming even the simplest things
again you may discover something that your predecessors did not, such as
creating code that can be faster while achieving the same task, or better
ways to network data between computers, the discoveries can be endless.
Even if no new breakthroughs have been achieved, that only solidifies the
current understanding of any subject. In this paper I showed the complexity
of creating the games so many people enjoy, on the way I learned more that
I could ever imagined
