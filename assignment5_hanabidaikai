#include <windows.h>				// Header File For Windows
#include <stdio.h>				// Header File For Standard Input/Output
#include <math.h>
#include <gl\gl.h>				// Header File For The OpenGL32 Library
#include <gl\glu.h>				// Header File For The GLu32 Library
#include <gl\glaux.h>			// Header File For The Glaux Library
#include <mmsystem.h>    	// PlaySound()
#pragma comment (lib, "winmm.lib") //PlaySound

HDC         hDC = NULL;			// Private GDI Device Context
HGLRC       hRC = NULL;			// Permanent Rendering Context
HWND        hWnd = NULL;		// Holds Our Window Handle
HINSTANCE   hInstance;			// Holds The Instance Of The Application

bool	keys[256];					// Array Used For The Keyboard Routine
bool	active = TRUE;				// Window Active Flag Set To TRUE By Default
bool	fullscreen = TRUE;		// Fullscreen Flag Set To Fullscreen Mode By Default
bool	rainbow = true;			// Rainbow Mode?
bool	sp;							// Spacebar Pressed?
bool	rp;							// Enter Key Pressed?

float	slowdown = 0.06f;//2.0f;// Slow Down Particles
float	xspeed;						// Base X Speed (To Allow Keyboard Direction Of Tail)
float	yspeed;						// Base Y Speed (To Allow Keyboard Direction Of Tail)
float	zoom = -40.0f;				// Used To Zoom Out

GLuint	col;						// Current Color Selection
GLuint	delay;					// Rainbow Effect Delay
GLuint	texture[2];				// Storage For Our Particle Texture

const int MAX_PARTICLES = 150; // 烟花散开的小烟花个数
const int MAX_TAIL = 50;      // 烟花尾部长度
const int MAX_FIRE = 5;       // 烟花最多个数

typedef struct {
   float r, g, b;      /* color */
   float x, y, z;      /* position  */
   float xs, ys, zs;   /* speed  */
   float xg, yg, zg;   /* gravity  */
   boolean up;         /* up or down */
} Particle;

typedef struct {
   Particle particle[MAX_PARTICLES][MAX_TAIL]; // 烟花系统数组
   float life, fade, rad; // 生命，衰减速度，x-z平面上的运动速度
} Fire;

Fire fire[MAX_FIRE];

static GLfloat colors[12][3]=		// Rainbow Of Colors
{
	{1.0f,0.5f,0.5f},{1.0f,0.75f,0.5f},{1.0f,1.0f,0.5f},{0.75f,1.0f,0.5f},
	{0.5f,1.0f,0.5f},{0.5f,1.0f,0.75f},{0.5f,1.0f,1.0f},{0.5f,0.75f,1.0f},
	{0.5f,0.5f,1.0f},{0.75f,0.5f,1.0f},{1.0f,0.5f,1.0f},{1.0f,0.5f,0.75f}
};

LRESULT	CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);	// Declaration For WndProc

AUX_RGBImageRec *LoadBMP(char *Filename){
FILE *File=NULL;
if(!Filename)
{return NULL;}
File=fopen(Filename,"r");//尝试打开文件
if(File){fclose(File); 
return auxDIBImageLoad(Filename);}
return NULL;
}

int LoadGLTextures(){
bool Status=FALSE; 
AUX_RGBImageRec *TextureImage[2];//创建纹理的存储空间
memset(TextureImage,0,sizeof(void *)*2);//将指针设为空
if(TextureImage[0]=LoadBMP("C:\\Particle.bmp")){
	Status=TRUE;
	glGenTextures(1,&texture[0]);//创建纹理

	//创建Nearest滤波贴图（低质量贴图）
	glBindTexture(GL_TEXTURE_2D,texture[0]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_NEAREST);//绘制的图片比贴图小
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_NEAREST);//绘制的图片比贴图大
	glTexImage2D(GL_TEXTURE_2D,0,3,TextureImage[0]->sizeX,TextureImage[0]->sizeY,0,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[0]->data);

	//创建线性滤波纹理
	glBindTexture(GL_TEXTURE_2D,texture[0]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
	glTexImage2D(GL_TEXTURE_2D,0,3,TextureImage[0]->sizeX,TextureImage[0]->sizeY,0,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[0]->data);

	//创建MipMapped纹理(可以绕过对纹理图片宽度和高度的限制)
	glBindTexture(GL_TEXTURE_2D,texture[0]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR_MIPMAP_NEAREST);
	gluBuild2DMipmaps(GL_TEXTURE_2D,3,TextureImage[0]->sizeX,TextureImage[0]->sizeY,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[0]->data);
}

if(TextureImage[1]=LoadBMP("C:\\hanabi1.bmp")){
	Status=TRUE;
	glGenTextures(1,&texture[1]);//创建纹理

	//创建Nearest滤波贴图（低质量贴图）
	glBindTexture(GL_TEXTURE_2D,texture[1]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_NEAREST);//绘制的图片比贴图小
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_NEAREST);//绘制的图片比贴图大
	glTexImage2D(GL_TEXTURE_2D,0,3,TextureImage[1]->sizeX,TextureImage[1]->sizeY,0,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[1]->data);

	//创建线性滤波纹理
	glBindTexture(GL_TEXTURE_2D,texture[1]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
	glTexImage2D(GL_TEXTURE_2D,0,3,TextureImage[1]->sizeX,TextureImage[1]->sizeY,0,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[1]->data);

	//创建MipMapped纹理(可以绕过对纹理图片宽度和高度的限制)
	glBindTexture(GL_TEXTURE_2D,texture[1]);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
	glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR_MIPMAP_NEAREST);
	gluBuild2DMipmaps(GL_TEXTURE_2D,3,TextureImage[1]->sizeX,TextureImage[1]->sizeY,GL_RGB,GL_UNSIGNED_BYTE,TextureImage[1]->data);
}

	if(TextureImage[0]){
		if(TextureImage[0]->data){
	free(TextureImage[0]->data);}
	free(TextureImage[0]);}

	if(TextureImage[1]){
		if(TextureImage[1]->data){
	free(TextureImage[1]->data);}
	free(TextureImage[1]);}

	return Status;
}
GLvoid ReSizeGLScene(GLsizei width, GLsizei height) // Resize And Initialize The GL Window
{
   if (height == 0) {								// Prevent A Divide By Zero By
      height = 1;										// Making Height Equal One
	}

	glViewport(0, 0, width, height);				// Reset The Current Viewport
	glMatrixMode(GL_PROJECTION);					// Select The Projection Matrix
	glLoadIdentity();									// Reset The Projection Matrix

	// Calculate The Aspect Ratio Of The Window
	gluPerspective(45.0f,(GLfloat)width/(GLfloat)height,0.1f,200.0f);

	glMatrixMode(GL_MODELVIEW);					// Select The Modelview Matrix
	glLoadIdentity();									// Reset The Modelview Matrix
   glTranslatef(0.0f, 0.0f, 0.0f);
   glRotatef(30.0f, 1.0f, 0.0f, 0.0f);
}

void resetFire(int loop) {
   // 坐标
   float xtemp = rand()%30 - 15.f;
   float ytemp = -1*rand()%5 - 15.f;//8.f;
   float ztemp = -1*rand()%5 - 15.f;//100.f;
   float speed = rand()%5 + 15.f;

   fire[loop].life = 1.5f;//1.0f;
   fire[loop].fade = (float) ((rand()%100)/20000 + 0.002);
   fire[loop].rad  = rand()%3 + 4.0f;

   for (int loop1 = 0; loop1 < MAX_PARTICLES; loop1++) {
      Particle* pat = &fire[loop].particle[loop1][0];
      //初始颜色
      pat->r = 1.0f; pat->g = 1.0f; pat->b = 1.0f;
      //初始位置
      pat->x = xtemp; pat->y = ytemp; pat->z = ztemp;
      //初始速度
      pat->xs = 0.0f; pat->ys = speed; pat->zs = 0.0f;
      //初始加速度
      pat->xg = 0.0f; pat->yg = -5.f; pat->zg = 0.0f;
      pat->up = true;

      //尾部初始化
      for(int loop2 = 1; loop2 < MAX_TAIL; loop2++) {
         pat = &fire[loop].particle[loop1][loop2];
         pat->x = fire[loop].particle[loop1][0].x;
         pat->y = fire[loop].particle[loop1][0].y;
         pat->z = fire[loop].particle[loop1][0].z;
      } //for loop2 end
   }//for loop1 end
}

int InitGL(GLvoid)					// All Setup For OpenGL Goes Here
{
   if (!LoadGLTextures()) {		// Jump To Texture Loading Routine
		return FALSE;					// If Texture Didn't Load Return FALSE
	}

	glShadeModel(GL_SMOOTH);							// Enable Smooth Shading
	glClearColor(0.0f,0.0f,0.0f,0.0f);					// Black Background
	glClearDepth(1.0f);									// Depth Buffer Setup
	glDisable(GL_DEPTH_TEST);							// Disable Depth Testing
	glEnable(GL_BLEND);									// Enable Blending
	glBlendFunc(GL_SRC_ALPHA,GL_ONE);					// Type Of Blending To Perform
	glHint(GL_PERSPECTIVE_CORRECTION_HINT,GL_NICEST);	// Really Nice Perspective Calculations
	glHint(GL_POINT_SMOOTH_HINT,GL_NICEST);				// Really Nice Point Smoothing
	glEnable(GL_TEXTURE_2D);							// Enable Texture Mapping
	glBindTexture(GL_TEXTURE_2D,texture[0]);			// Select Our Texture

   for(int loop = 0; loop < MAX_FIRE; loop++) {
      resetFire(loop);
   }//for loop end

	return TRUE;										// Initialization Went OK
}

int DrawGLScene(GLvoid)	{ // Here's Where We Do All The Drawing
   glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);		// Clear Screen And Depth Buffer
   glLoadIdentity();		// Reset The ModelView Matrix

   float rgb_max_value = 0.0f;
   for(int loop0 = 0; loop0 < MAX_FIRE; loop0++) {
      float rgb_value = fire[loop0].particle[0][0].r + fire[loop0].particle[0][0].g + fire[loop0].particle[0][0].b;
      if (rgb_value > rgb_max_value) {
         rgb_max_value = rgb_value;
         glColor4f(fire[loop0].particle[0][0].r, fire[loop0].particle[0][0].g, fire[loop0].particle[0][0].b, 1.0f);
      }
   }

   glBindTexture(GL_TEXTURE_2D, texture[1]);
   glBegin(GL_TRIANGLE_STRIP);  // Build Quad From A Triangle Strip
   float w = 0.05522847498307933984022516322796f, h = 0.04142135623730950488016887242097f;
   // h = tan(45.f/2) * 0.1f; w = 640.f/480.f*h;
   glTexCoord2d(1,1); glVertex3f(w, h, -0.1f); // Top Right
   glTexCoord2d(0,1); glVertex3f(-w, h, -0.1f); // Top Left
   glTexCoord2d(1,0); glVertex3f(w, -h, -0.1f); // Bottom Right
   glTexCoord2d(0,0); glVertex3f(-w, -h, -0.1f); // Bottom Left
   glEnd(); // Done Building Triangle Strip

   glBindTexture(GL_TEXTURE_2D, texture[0]);
   for(int loop = 0; loop < MAX_FIRE; loop++) {
      int color = rand()%12;
      for (int loop1 = 0; loop1 < MAX_PARTICLES; loop1++) {
         for(int loop2 = 0; loop2 < MAX_TAIL; loop2++) {
            float x = fire[loop].particle[loop1][loop2].x;
            float y = fire[loop].particle[loop1][loop2].y;
            float z = fire[loop].particle[loop1][loop2].z + zoom;
            float dt = 1 - (float)loop2/MAX_TAIL;
            glColor4f(fire[loop].particle[loop1][0].r, 
               fire[loop].particle[loop1][0].g, 
               fire[loop].particle[loop1][0].b, 
               fire[loop].life * dt);

            float size = 0.5f * dt;
            glBegin(GL_TRIANGLE_STRIP);   // Build Quad From A Triangle Strip
            glTexCoord2d(1,1); glVertex3f(x+size,y+size,z); // Top Right
            glTexCoord2d(0,1); glVertex3f(x-size,y+size,z); // Top Left
            glTexCoord2d(1,0); glVertex3f(x+size,y-size,z); // Bottom Right
            glTexCoord2d(0,0); glVertex3f(x-size,y-size,z); // Bottom Left
            glEnd(); 
         }

         // 位置更新
         for(int loop3 = MAX_TAIL-1; loop3 > 0; loop3--) {
            fire[loop].particle[loop1][loop3].x = fire[loop].particle[loop1][loop3-1].x;
            fire[loop].particle[loop1][loop3].y = fire[loop].particle[loop1][loop3-1].y;
            fire[loop].particle[loop1][loop3].z = fire[loop].particle[loop1][loop3-1].z;
         } //for loop3 end

         fire[loop].particle[loop1][0].x += fire[loop].particle[loop1][0].xs / (slowdown * 1000);
         fire[loop].particle[loop1][0].y += fire[loop].particle[loop1][0].ys / (slowdown * 1000);
         fire[loop].particle[loop1][0].z += fire[loop].particle[loop1][0].zs / (slowdown * 1000);

         //y速度更新
         float yd = fire[loop].particle[loop1][0].yg / (slowdown * 1000);
         fire[loop].particle[loop1][0].ys += yd;
         if (fire[loop].particle[loop1][0].up && fire[loop].particle[loop1][0].ys < -yd) {
            fire[loop].particle[loop1][0].up = false;
            //int color = rand()%12;
            fire[loop].particle[loop1][0].r = colors[color][0];
            fire[loop].particle[loop1][0].g = colors[color][1];
            fire[loop].particle[loop1][0].b = colors[color][2];

            // x-z 的速度
            double radian = 3.14159*loop1*15.0/180.0;
            fire[loop].particle[loop1][0].xs = (float)(fire[loop].rad*sin(radian));
            fire[loop].particle[loop1][0].zs = (float)(fire[loop].rad*cos(radian));
         }

         if (keys[VK_UP] && fire[loop].particle[loop1][0].yg < 1.5f) {
            fire[loop].particle[loop1][0].yg += 0.01f;
         }

         if (keys[VK_DOWN] && fire[loop].particle[loop1][0].yg > -1.5f) {
            fire[loop].particle[loop1][0].yg -= 0.01f;
         }

         if (keys[VK_LEFT] && fire[loop].particle[loop1][0].xg < 1.5f) {
            fire[loop].particle[loop1][0].yg += 0.01f;
         }

         if (keys[VK_RIGHT] && fire[loop].particle[loop1][0].xg > -1.5f) {
            fire[loop].particle[loop1][0].yg -= 0.01f;
         }
      }

      fire[loop].life -= fire[loop].fade;
      if (fire[loop].life < 0) {
         resetFire(loop);
      }
   }
   
   if (keys[VK_TAB])	{ // Tab Key Causes A Burst
      for (int loop = 0; loop < MAX_FIRE; loop++) {
         resetFire(loop);
      }
   }
   return TRUE;
}

// Properly Kill The Window
GLvoid KillGLWindow(GLvoid) {
   if (fullscreen) {							// Are We In Fullscreen Mode?
		ChangeDisplaySettings(NULL,0);	// If So Switch Back To The Desktop
		ShowCursor(TRUE);						// Show Mouse Pointer
	}

   if (hRC)	{ // Do We Have A Rendering Context?
      if (!wglMakeCurrent(NULL,NULL)) {   // Are We Able To Release The DC And RC Contexts?
         MessageBox(NULL,"Release Of DC And RC Failed.","SHUTDOWN ERROR",MB_OK | MB_ICONINFORMATION);
		}

      if (!wglDeleteContext(hRC)) { // Are We Able To Delete The RC?
			MessageBox(NULL,"Release Rendering Context Failed.","SHUTDOWN ERROR",MB_OK | MB_ICONINFORMATION);
		}
		hRC = NULL;								// Set RC To NULL
	}

   if (hDC && !ReleaseDC(hWnd,hDC))	{	// Are We Able To Release The DC
		MessageBox(NULL,"Release Device Context Failed.","SHUTDOWN ERROR",MB_OK | MB_ICONINFORMATION);
		hDC = NULL;								// Set DC To NULL
	}

   if (hWnd && !DestroyWindow(hWnd)) { // Are We Able To Destroy The Window?
		MessageBox(NULL,"Could Not Release hWnd.","SHUTDOWN ERROR",MB_OK | MB_ICONINFORMATION);
		hWnd = NULL;							// Set hWnd To NULL
	}

   if (!UnregisterClass("OpenGL",hInstance))	{	// Are We Able To Unregister Class
		MessageBox(NULL,"Could Not Unregister Class.","SHUTDOWN ERROR",MB_OK | MB_ICONINFORMATION);
		hInstance = NULL;									// Set hInstance To NULL
	}
}

/*	This Code Creates Our OpenGL Window.  Parameters Are:					*
 *	title			- Title To Appear At The Top Of The Window				*
 *	width			- Width Of The GL Window Or Fullscreen Mode				*
 *	height			- Height Of The GL Window Or Fullscreen Mode			*
 *	bits			- Number Of Bits To Use For Color (8/16/24/32)			*
 *	fullscreenflag	- Use Fullscreen Mode (TRUE) Or Windowed Mode (FALSE)	*/
 
BOOL CreateGLWindow(char* title, int width, int height, int bits, bool fullscreenflag)
{
	GLuint	PixelFormat;		// Holds The Results After Searching For A Match
	WNDCLASS	wc;					// Windows Class Structure
	DWORD		dwExStyle;			// Window Extended Style
	DWORD		dwStyle;				// Window Style
	RECT		WindowRect;			// Grabs Rectangle Upper Left / Lower Right Values
	WindowRect.left   = (long)0;		// Set Left Value To 0
	WindowRect.right  = (long)width;	// Set Right Value To Requested Width
	WindowRect.top    = (long)0;		// Set Top Value To 0
	WindowRect.bottom = (long)height;// Set Bottom Value To Requested Height

	fullscreen = fullscreenflag;	// Set The Global Fullscreen Flag

	hInstance		= GetModuleHandle(NULL);		 // Grab An Instance For Our Window
	wc.style			= CS_HREDRAW | CS_VREDRAW | CS_OWNDC;	// Redraw On Size, And Own DC For Window.
	wc.lpfnWndProc	= (WNDPROC) WndProc;				 // WndProc Handles Messages
	wc.cbClsExtra	= 0;									 // No Extra Window Data
	wc.cbWndExtra	= 0;									 // No Extra Window Data
	wc.hInstance	= hInstance;						 // Set The Instance
	wc.hIcon			= LoadIcon(NULL, IDI_WINLOGO); // Load The Default Icon
	wc.hCursor		= LoadCursor(NULL, IDC_ARROW); // Load The Arrow Pointer
	wc.hbrBackground = NULL;							 // No Background Required For GL
	wc.lpszMenuName  = NULL;							 // We Don't Want A Menu
	wc.lpszClassName = "OpenGL";						 // Set The Class Name

   if (!RegisterClass(&wc)) {                   // Attempt To Register The Window Class
		MessageBox(NULL,"Failed To Register The Window Class.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;										// Return FALSE
	}

   if (fullscreen) {				 // Attempt Fullscreen Mode?
		DEVMODE dmScreenSettings;// Device Mode
		memset(&dmScreenSettings,0,sizeof(dmScreenSettings));	// Makes Sure Memory's Cleared
		dmScreenSettings.dmSize=sizeof(dmScreenSettings);	// Size Of The Devmode Structure
		dmScreenSettings.dmPelsWidth	= width;             // Selected Screen Width
		dmScreenSettings.dmPelsHeight	= height;				// Selected Screen Height
		dmScreenSettings.dmBitsPerPel	= bits;					// Selected Bits Per Pixel
		dmScreenSettings.dmFields=DM_BITSPERPEL|DM_PELSWIDTH|DM_PELSHEIGHT;

		// Try To Set Selected Mode And Get Results.  NOTE: CDS_FULLSCREEN Gets Rid Of Start Bar.
		if (ChangeDisplaySettings(&dmScreenSettings,CDS_FULLSCREEN) != DISP_CHANGE_SUCCESSFUL) {
			// If The Mode Fails, Offer Two Options.  Quit Or Use Windowed Mode.
			if (MessageBox(NULL,"The Requested Fullscreen Mode Is Not Supported By\nYour Video Card. Use Windowed Mode Instead?","NeHe GL",MB_YESNO|MB_ICONEXCLAMATION)==IDYES) {
				fullscreen=FALSE;		// Windowed Mode Selected.  Fullscreen = FALSE
			} else {
				// Pop Up A Message Box Letting User Know The Program Is Closing.
				MessageBox(NULL,"Program Will Now Close.","ERROR",MB_OK|MB_ICONSTOP);
				return FALSE;									// Return FALSE
			}
		}
	}

   if (fullscreen) {                // Are We Still In Fullscreen Mode?
		dwExStyle = WS_EX_APPWINDOW;	// Window Extended Style
		dwStyle = WS_POPUP;				// Windows Style
		ShowCursor(FALSE);				// Hide Mouse Pointer
	} else {
		dwExStyle = WS_EX_APPWINDOW | WS_EX_WINDOWEDGE;	// Window Extended Style
		dwStyle = WS_OVERLAPPEDWINDOW;						// Windows Style
	}

	AdjustWindowRectEx(&WindowRect, dwStyle, FALSE, dwExStyle);		// Adjust Window To True Requested Size

	// Create The Window
	if (!(hWnd=CreateWindowEx(	dwExStyle,							// Extended Style For The Window
								"OpenGL",							// Class Name
								title,								// Window Title
								dwStyle |							// Defined Window Style
								WS_CLIPSIBLINGS |					// Required Window Style
								WS_CLIPCHILDREN,					// Required Window Style
								0, 0,								// Window Position
								WindowRect.right-WindowRect.left,	// Calculate Window Width
								WindowRect.bottom-WindowRect.top,	// Calculate Window Height
								NULL,								// No Parent Window
								NULL,								// No Menu
								hInstance,							// Instance
								NULL)))								// Dont Pass Anything To WM_CREATE
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Window Creation Error.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	static	PIXELFORMATDESCRIPTOR pfd=				// pfd Tells Windows How We Want Things To Be
	{
		sizeof(PIXELFORMATDESCRIPTOR),				// Size Of This Pixel Format Descriptor
		1,											// Version Number
		PFD_DRAW_TO_WINDOW |						// Format Must Support Window
		PFD_SUPPORT_OPENGL |						// Format Must Support OpenGL
		PFD_DOUBLEBUFFER,							// Must Support Double Buffering
		PFD_TYPE_RGBA,								// Request An RGBA Format
		bits,										// Select Our Color Depth
		0, 0, 0, 0, 0, 0,							// Color Bits Ignored
		0,											// No Alpha Buffer
		0,											// Shift Bit Ignored
		0,											// No Accumulation Buffer
		0, 0, 0, 0,									// Accumulation Bits Ignored
		16,											// 16Bit Z-Buffer (Depth Buffer)  
		0,											// No Stencil Buffer
		0,											// No Auxiliary Buffer
		PFD_MAIN_PLANE,								// Main Drawing Layer
		0,											// Reserved
		0, 0, 0										// Layer Masks Ignored
	};
	
	if (!(hDC=GetDC(hWnd)))							// Did We Get A Device Context?
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Can't Create A GL Device Context.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	if (!(PixelFormat=ChoosePixelFormat(hDC,&pfd)))	// Did Windows Find A Matching Pixel Format?
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Can't Find A Suitable PixelFormat.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	if(!SetPixelFormat(hDC,PixelFormat,&pfd))		// Are We Able To Set The Pixel Format?
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Can't Set The PixelFormat.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	if (!(hRC=wglCreateContext(hDC)))				// Are We Able To Get A Rendering Context?
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Can't Create A GL Rendering Context.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	if(!wglMakeCurrent(hDC,hRC))					// Try To Activate The Rendering Context
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Can't Activate The GL Rendering Context.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	ShowWindow(hWnd,SW_SHOW);						// Show The Window
	SetForegroundWindow(hWnd);						// Slightly Higher Priority
	SetFocus(hWnd);									// Sets Keyboard Focus To The Window
	ReSizeGLScene(width, height);					// Set Up Our Perspective GL Screen

	if (!InitGL())									// Initialize Our Newly Created GL Window
	{
		KillGLWindow();								// Reset The Display
		MessageBox(NULL,"Initialization Failed.","ERROR",MB_OK|MB_ICONEXCLAMATION);
		return FALSE;								// Return FALSE
	}

	return TRUE;									// Success
}

LRESULT CALLBACK WndProc(	HWND	hWnd,			// Handle For This Window
							UINT	uMsg,			// Message For This Window
							WPARAM	wParam,			// Additional Message Information
							LPARAM	lParam)			// Additional Message Information
{
	switch (uMsg)									// Check For Windows Messages
	{
   case WM_ACTIVATE:	{     					// Watch For Window Activate Message
	   			
	  //PlaySound(("C:\\hanabidaikai.wav"),NULL,SND_LOOP | SND_ASYNC);	

      if (!HIWORD(wParam)) {				// Check Minimization State
			active=TRUE;						// Program Is Active
		} else {									// Otherwise
			active=FALSE;						// Program Is No Longer Active
		}
		return 0;								// Return To The Message Loop
   }

   case WM_SYSCOMMAND: {						// Intercept System Commands
		switch (wParam) { 						// Check System Calls
			case SC_SCREENSAVE:					// Screensaver Trying To Start?
			case SC_MONITORPOWER:				// Monitor Trying To Enter Powersave?
			return 0;							// Prevent From Happening
		}
		break;									// Exit
	}
   
   case WM_CLOSE:	{							// Did We Receive A Close Message?
		PostQuitMessage(0);					// Send A Quit Message
		return 0;								// Jump Back
	}

   case WM_KEYDOWN: {						// Is A Key Being Held Down?
		keys[wParam] = TRUE;					// If So, Mark It As TRUE
		return 0;								// Jump Back
	}

   case WM_KEYUP: {						// Has A Key Been Released?
      keys[wParam] = FALSE;		// If So, Mark It As FALSE
		return 0;						// Jump Back
	}

	case WM_SIZE: {					// Resize The OpenGL Window
		ReSizeGLScene(LOWORD(lParam),HIWORD(lParam));  // LoWord=Width, HiWord=Height
		return 0;								// Jump Back
	}
   }
	// Pass All Unhandled Messages To DefWindowProc
	return DefWindowProc(hWnd,uMsg,wParam,lParam);
}

int WINAPI WinMain(HINSTANCE	hInstance,			// Instance
					HINSTANCE   hPrevInstance,		// Previous Instance
					LPSTR       lpCmdLine,			// Command Line Parameters
					int         nCmdShow)			// Window Show State
{
	MSG		msg;									// Windows Message Structure
	BOOL	done = FALSE;								// Bool Variable To Exit Loop
   char* title = "花火大会";

	// Ask The User Which Screen Mode They Prefer
	if (MessageBox(NULL,"Would You Like To Run In Fullscreen Mode?", "Start FullScreen?",MB_YESNO|MB_ICONQUESTION)==IDNO) {
      fullscreen = FALSE;
	}

	if (!CreateGLWindow(title, 640, 480, 16, fullscreen)) {
      return 0;
	}

   if (fullscreen) {								// Are We In Fullscreen Mode
      slowdown /= 2.0f;							// If So, Speed Up The Particles (3dfx Issue)
   }

   // Loop That Runs While done=FALSE
	while (!done) {
      if (PeekMessage(&msg,NULL,0,0,PM_REMOVE))	{ // Is There A Message Waiting?
         if (msg.message == WM_QUIT) { // Have We Received A Quit Message?
				done = TRUE;					// If So done=TRUE
			} else {                      // If Not, Deal With Window Messages
				TranslateMessage(&msg);		// Translate The Message
				DispatchMessage(&msg);		// Dispatch The Message
			}
		}
		else  // If There Are No Messages
		{
			// Draw The Scene.  Watch For ESC Key And Quit Messages From DrawGLScene()
			if ((active && !DrawGLScene()) || keys[VK_ESCAPE])	{ // Active?  Was There A Quit Received?
				done = TRUE; // ESC or DrawGLScene Signalled A Quit
			}
         else { // Not Time To Quit, Update Screen
				SwapBuffers(hDC);	// Swap Buffers (Double Buffering)

				if (keys[VK_ADD] && slowdown > 1.0f) slowdown -= 0.01f;		// Speed Up Particles
				if (keys[VK_SUBTRACT] && slowdown < 4.0f) slowdown += 0.01f;// Slow Down Particles

				if (keys[VK_PRIOR])	zoom+=0.01f;		// Zoom In
				if (keys[VK_NEXT])	zoom-=0.01f;		// Zoom Out

            if (keys[VK_RETURN] && !rp)	{	// Return Key Pressed
					rp = true;					// Set Flag Telling Us It's Pressed
					rainbow = !rainbow;		// Toggle Rainbow Mode On / Off
				}
				
            if (!keys[VK_RETURN]) rp = false;		// If Return Is Released Clear Flag

            if ((keys[' '] && !sp) || (rainbow && (delay > 25))) { // Space Or Rainbow Mode
					if (keys[' '])	rainbow = false;	// If Spacebar Is Pressed Disable Rainbow Mode
					sp = true;						// Set Flag Telling Us Space Is Pressed
					delay = 0;						// Reset The Rainbow Color Cycling Delay
					col++;							// Change The Particle Color
					if (col > 11)	col = 0;				// If Color Is To High Reset It
				}
				if (!keys[' ']) sp = false;			// If Spacebar Is Released Clear Flag

				// If Up Arrow And Y Speed Is Less Than 200 Increase Upward Speed
				if (keys[VK_UP] && (yspeed<200)) yspeed+=0.5f;

				// If Down Arrow And Y Speed Is Greater Than -200 Increase Downward Speed
				if (keys[VK_DOWN] && (yspeed>-200)) yspeed-=0.5f;

				// If Right Arrow And X Speed Is Less Than 200 Increase Speed To The Right
				if (keys[VK_RIGHT] && (xspeed<200)) xspeed+=0.5f;

				// If Left Arrow And X Speed Is Greater Than -200 Increase Speed To The Left
				if (keys[VK_LEFT] && (xspeed>-200)) xspeed-=0.5f;

				delay++;							// Increase Rainbow Mode Color Cycling Delay Counter

				if (keys[VK_F1]) {			// Is F1 Being Pressed?
					keys[VK_F1] = FALSE;		// If So Make Key FALSE
					KillGLWindow();			// Kill Our Current Window
					fullscreen = !fullscreen;	// Toggle Fullscreen / Windowed Mode
					// Recreate Our OpenGL Window
					if (!CreateGLWindow(title, 640, 480, 16, fullscreen)) {
						return 0;	// Quit If Window Was Not Created
					}
				}
			}
		}
	}
	// Shutdown
	KillGLWindow();		// Kill The Window
	return (msg.wParam);	// Exit The Program
}
