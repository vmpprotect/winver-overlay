# winver-overlay
also works for notepad.exe

# Hijacking Window

```cpp
    inline HWND HiJackNotepadWindow() {
        HWND hwnd = NULL;

        // Remove previous windows
        vector<DWORD> existingNotepads = GetPIDs((L"winver.exe"));
        if (!existingNotepads.empty()) {
            for (int i(0); i < existingNotepads.size(); ++i) {
                // Terminating processes
                HANDLE hOldProcess = OpenProcess(PROCESS_TERMINATE, FALSE, existingNotepads[i]);
                TerminateProcess(hOldProcess, 0);
                CloseHandle(hOldProcess);
            }
        }

        system(("start winver.exe")); // Start notepad, and not as child process, so easy :)

        // Finding notepad's window (we could just use FindWindow but then it would be OS language dependent)
        vector<DWORD> notepads = GetPIDs((L"winver.exe"));
        if (notepads.empty() || notepads.size() > 1) // Should check if more than one to be more strict
            return hwnd;
        WindowsFinderParams params;
        params.pidOwner = notepads[0];
        params.style = WS_VISIBLE;
        params.satisfyAllCriteria = true;
        vector<HWND> hwnds;
        int attempt = 0; // The process takes a bit of time to initialise and spawn the window, will try during 5 sec before time out
        while (hwnd == NULL || attempt > 50) {
            Sleep(7500);
            hwnds = WindowsFinder(params);
            if (hwnds.size() > 1)
                return hwnd;
            hwnd = hwnds[0];
            ++attempt;
        }
        if (!hwnd)
            return hwnd;

        // Making the window usable for overlay puposes

        SetMenu(hwnd, NULL);
        SetWindowLongPtr(hwnd, GWL_STYLE, WS_VISIBLE);
        SetWindowLongPtr(hwnd, GWL_EXSTYLE, WS_EX_LAYERED | WS_EX_TRANSPARENT | WS_EX_NOACTIVATE | WS_EX_TOOLWINDOW); // WS_EX_NOACTIVATE  and WS_EX_TOOLWINDOW removes it from taskbar

        SetWindowPos(hwnd, NULL, 0, 0, GetSystemMetrics(SM_CXSCREEN), GetSystemMetrics(SM_CYSCREEN), SWP_SHOWWINDOW);
        screen_width = GetSystemMetrics(SM_CXSCREEN);
        screen_height = GetSystemMetrics(SM_CYSCREEN);
        return hwnd;
    }
```

## in render
```
        auto hijack() -> bool
        {
            while (!window_handle) {
                window_handle = SetUp::HiJackNotepadWindow();
                Sleep(10);
            }
            MARGINS margin = { -1 };
            DwmExtendFrameIntoClientArea(window_handle, &margin);
            SetMenu(window_handle, NULL);
            SetWindowLongPtr(window_handle, GWL_STYLE, WS_VISIBLE);
            SetWindowLongPtr(window_handle, GWL_EXSTYLE, WS_EX_LAYERED | WS_EX_TRANSPARENT);

            ShowWindow(window_handle, SW_SHOW);
            UpdateWindow(window_handle);

            SetWindowLong(window_handle, GWL_EXSTYLE, GetWindowLong(window_handle, GWL_EXSTYLE) | WS_EX_LAYERED | WS_EX_TRANSPARENT);

            return true;

            if (misc::Streamproof) {
                SetWindowDisplayAffinity(window_handle, WDA_EXCLUDEFROMCAPTURE);
            }
            else {
                SetWindowDisplayAffinity(window_handle, !WDA_EXCLUDEFROMCAPTURE);
            }

            /*window = FindWindowA(_("MedalOverlayClass"), _("MedalOverlay"));
            if (!window)
            {
                MessageBoxA(NULL, "Medal Not Found", "Overlay", ALERT_SYSTEM_WARNING);
                return false;
            }
            MARGINS Margin = { -1 };
            DwmExtendFrameIntoClientArea(window, &Margin);
            SetWindowPos(window, 0, 0, 0, GetSystemMetrics(SM_CXSCREEN), GetSystemMetrics(SM_CYSCREEN), SWP_SHOWWINDOW);
            ShowWindow(window, SW_SHOW);
            UpdateWindow(window);
            return true;*/
            // window_handle = FindWindowA_Spoofed(E("CiceroUIWndFrame"), E("CiceroUIWndFrame")); // visual studio
           //  window_handle = FindWindowA_Spoofed ( E ("MedalOverlayClass" ), E ( "MedalOverlay" ) ); // Medal
           // window_handle = FindWindowA_Spoofed(E("CEF-OSC-WIDGET"), E("NVIDIA Geforce Overlay")); // Nvidia
            //  window_handle = FindWindowA_Spoofed(E("GDI+ Hook Class"), E("GDI+ Window (FPSMonitor)")); // Monitor
             // window_handle = FindWindowA_Spoofed(E("GDI+ Hook Window Class"), E("GDI+ Window (obs64.exe)")); // obs





 /*           if (!window_handle)
            {

                return false;
            }

            int WindowWidth = GetSystemMetrics(SM_CXSCREEN);
            int WindowHeight = GetSystemMetrics(SM_CYSCREEN);

            if (!SetWindowPos_Spoofed(window_handle, HWND_TOPMOST, 0, 0, WindowWidth, WindowHeight, SWP_SHOWWINDOW))
            {
                return false;
            }

            if (!SetLayeredWindowAttributes_Spoofed(window_handle, RGB(0, 0, 0), 255, LWA_ALPHA))
            {
                return false;
            }

            if (!SetWindowLongA_Spoofed(window_handle, GWL_EXSTYLE, GetWindowLong(window_handle, GWL_EXSTYLE) | WS_EX_TRANSPARENT))
            {
                return false;
            }

            MARGINS Margin = { -1 };
            if (FAILED(DwmExtendFrameIntoClientArea(window_handle, &Margin)))
            {
                return false;
            }

            ShowWindow_Spoofed(window_handle, SW_SHOW);

            UpdateWindow_Spoofed(window_handle);

            ShowWindow_Spoofed(window_handle, SW_HIDE);
            if (misc::Streamproof) {
                SetWindowDisplayAffinity(window_handle, !WDA_EXCLUDEFROMCAPTURE); //UD
            }
            return true;*/
        }

```
