---
title: "WinForm 內部對話框，標題拖曳功能"
date: 2020-11-20T20:12:59+08:00
draft: false
tags: 
  - WinForm
  - WindowsAPI
description: "開發 WinForm 時，要做到WinForm內部對話框，而且又可以按住標題拖曳的效果，可以透過Panel手刻出對話框，透過WinAPI做到拖曳效果。"
---
開發 WinForm 時，要做到WinForm內部對話框，而且又可以按住標題拖曳的效果，可以透過`Panel`手刻出對話框，透過WinAPI做到拖曳效果。

``` cs
public partial class Form1 : Form
{
    [DllImport("user32.dll")]
    public static extern bool ReleaseCapture();
    [DllImport("user32.dll")]
    public static extern bool SendMessage(IntPtr hwnd, int wMsg, int wParam, int lParam);

    public Form1()
    {
        InitializeComponent();
    }

    private void DialogTitle_MouseDown(object sender, MouseEventArgs e)
    {
        if (e.Button == MouseButtons.Left)
        {
            //釋放鼠標捕捉
            ReleaseCapture(); 

            //發送左鍵點擊的消息至該視窗
            SendMessage(FrocePanel1.Handle, 0xA1, 0x02, 0);
        }
    }
}
```

[範例程式碼](https://github.com/patrick85081/WinForm_Dialog_Drag)
