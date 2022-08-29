# Windows编程



主函数

```c
WinMain( _In_ HINSTANCE hInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ LPSTR lpCmdLine, _In_ int nShowCmd )
```

静态库(lib)和动态库(dll)

静态库在程序运行的时候会直接内嵌进程序代码里面

动态库在程序运行时只是提供一个地址，动态生成函数

句柄都是找内存的，通过把窗口

<img src="C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220811124913470.png" alt="image-20220811124913470" style="zoom:67%;" />

windows位于用户接口程序和应用程序之间(两种的功能都具备)



## 1.创建一个窗口

```C

#include<windows.h>

//消息处理函数
LRESULT CALLBACK WndProc(HWND hwnd, UINT msgID, WPARAM wparam, LPARAM lparam)
{
	switch (msgID)
	{

	}

	return DefWindowProc(hwnd, msgID, wparam, lparam);
}



int WINAPI WinMain( _In_ HINSTANCE hInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ LPSTR lpCmdLine, _In_ int nShowCmd )
{
	//设计窗口类，并注册
	WNDCLASS wndclass = { 0 };
	//设计窗口类，即给WNDCLASS这个结构体的10个成员赋值
	//窗口类的附加内存，除系统固定分配的内存外，还可以额外给它一些内存
	wndclass.cbClsExtra = 0;
	//窗口的附加内存
	wndclass.cbWndExtra = 0;
	//设置窗口背景
	wndclass.hbrBackground = (HGDIOBJ)GetStockObject(WHITE_BRUSH);
	//鼠标指针样式
	wndclass.hCursor = NULL;
	//图标
	wndclass.hIcon = NULL;
    //程序的实例句柄
	wndclass.hInstance = hInstance;
	//程序处理消息的时候调用的是哪一个函数
	wndclass.lpfnWndProc = WndProc;
	//给类起个名
	wndclass.lpszClassName = TEXT("MainWindow");
	//菜单名 NULL为不要菜单
	wndclass.lpszMenuName = NULL;
	//窗口的样式  平面移动则刷新页面 
	wndclass.style = CS_HREDRAW | CS_VREDRAW;
	
	//注册到系统当中，把wndclass相关数据写入内核当中
	RegisterClass(&wndclass);
		
	//创建窗口 用类实例化一个窗口  第一个参数LpwndClassName指定创建的窗口时基于
    //哪一个窗口类的
	HWND hwnd = CreateWindow(TEXT("MainWindow"), TEXT("MainWindow"), WS_OVERLAPPEDWINDOW, 0, 0, 1024, 768, NULL, NULL, hInstance, NULL);

	//显示窗口, 更新窗口
	ShowWindow(hwnd, SW_SHOW);
	UpdateWindow(hwnd);
	
	//消息循环
	MSG msg = { 0 };
	//GetMessage会只抓取指定窗口的消息，所以对于整个程序，不能设定指定窗口
	while (GetMessage(&msg, NULL, 0, 0))
	{

		TranslateMessage(&msg);//翻译消息

		DispatchMessage(&msg);//派发消息, 之后会触发处理消息

	}

	//MessageBox(NULL, "艾佳微彩笔", "提示", MB_ICONERROR | MB_ABORTRETRYIGNORE);
	
	return 0;
}
```









```c
typedef struct tagMSG {
    HWND        hwnd;//发出消息的那个窗口的句柄
    UINT        message;//接受的消息
    //消息的附加信息，告诉程序是具体的什么信息
    WPARAM      wParam;
    LPARAM      lParam;
    DWORD       time;//消息产生的时间
    POINT       pt;//产生消息的位置
#ifdef _MAC
    DWORD       lPrivate;
#endif
} MSG, *PMSG, NEAR *NPMSG, FAR *LPMSG;
```



## Windows消息处理机制

![image-20220811152135283](C:\Users\Urizen\AppData\Roaming\Typora\typora-user-images\image-20220811152135283.png)





