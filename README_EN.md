# Campus Network Auto Login (校园网自动登录)

Auto-login for campus network web portals on Windows — works on both **wired Ethernet
and Wi-Fi** (any SSID), with **no popup windows** and **no polling**.

> Currently verified on **Shaanxi Normal University (SNNU)**. Other schools can be added
> by dropping a small *driver* file into `drivers/` — no code changes needed.

## Features

| | |
|---|---|
| Triggers | Windows logon · network connected (cable/Wi-Fi) · Wi-Fi connected · internet lost |
| Speed | ~25 ms offline detection via raw TCP probes (no DNS lookup), login verified in ~3 s |
| No popup | Launched through a hidden VBS wrapper |
| Portable | Account/password in a separate config file; one-click install & uninstall |
| Extensible | Portal logic described by data-only *driver* files (`.psd1`) |
| GUI | WinForms configuration tool (`gui.cmd`) |

## Quick start

1. Double-click `gui.cmd`.
2. Enter your account / password / carrier, tick **「记住账号密码（仅存本机）」**
   (remember credentials — stored locally only), click **测试登录** to verify,
   then **安装 / 更新任务**.
3. Done. It will auto-authenticate on logon, on cable/Wi-Fi connect, and whenever the
   internet drops.

> Prefer manual config? Copy `config.example.psd1` → `config.psd1`, fill it in, run `install.cmd`.
> Credentials are only kept in the local `config.psd1`, which is git-ignored and never uploaded.

No administrator rights required. No Python or other runtime needed — pure PowerShell 5.1
plus the built-in `curl.exe`.

## Adding your school

1. Copy `drivers/generic.psd1` to `drivers/<yourschool>.psd1`.
2. Log in once in a browser with DevTools (F12 → Network) and copy the login request's
   URL, method and form fields into the driver.
3. Set `driver = '<yourschool>'` in `config.psd1`.

Success is judged by *whether the internet actually works*, so you only need the request
itself to be correct.

## Files

```
campus-login.ps1        main script (generic, driver-driven)
config.psd1             your settings (git-ignored)
config.example.psd1     settings template
drivers/snnu.psd1       Shaanxi Normal University driver
drivers/generic.psd1    template for any simple form-POST portal
gui.ps1 / gui.cmd       graphical configuration tool
gui-run.ps1             launcher wrapper for the GUI (forces UTF-8 output)
install.ps1 / .cmd      register the scheduled task
uninstall.ps1 / .cmd    remove the scheduled task
status.ps1 / status.cmd health check (shows whether anything is running)
run-hidden.vbs          hidden-window launcher (no popup)
logs/                   runtime logs (git-ignored)
```

## License

[MIT](LICENSE)
