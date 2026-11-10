# Convoar Windows for Web_3d — complete setup guide

**English** | [Français](WINDOWS_WEB3D.fr.md)

> This is a fork of [Misterblue/convoar](https://github.com/Misterblue/convoar). The Web_3d Windows distribution **packages and tests the existing binary** checked into `dist/` with its dependencies. It **does not compile a new C# build**, and it does not install Python.

## 1. Download

1. Visit this fork's [Releases](https://github.com/Vlad-BloodMoon/convoar/releases). Download **Convoar-Windows-Web3D.zip** from **Assets**, not the automatically generated Source code archives.
2. Extract the entire ZIP, for example into `C:\Web3D\Convoar\`. Find `Convoar\convoar.exe`. Keep **all DLLs and configuration files together**.
3. Download and extract [Klaatou's Web_3d](https://github.com/Klaatou/Web_3d). Its `dist/oar2glb.py` is the conversion orchestration tool.
4. Have an OpenSimulator `.oar` backup available, and keep an untouched copy.
5. Windows requires **.NET Framework 4.8** for this Convoar binary (this is different from .NET 8 or later).

This workflow tests on Windows; other platforms have not been validated by that test.

## 2. Test Convoar in PowerShell

Open the extracted `Convoar` folder in File Explorer, enter `powershell` in the address bar, and run:

```powershell
.\convoar.exe --help
```

In PowerShell, **`.\` is necessary** for an executable in the current folder. Running `convoar.exe` without it may show “not recognized”. Seeing the options verifies startup, **not full OAR compatibility**.

## 3. Install Python on Windows for Web_3d

**Python is for Web_3d's `oar2glb.py`; the Convoar executable itself does not need Python.**

1. Download a maintained Windows 64-bit Python from [python.org](https://www.python.org/downloads/windows/) (Web_3d documents Python 3.9+). Our manual test succeeded with **Python 3.14.10**.
2. In the classic installer, check **Add python.exe to PATH**, choose **Install Now**, and note the suggested installation path.
3. **Close and reopen PowerShell**, then test:

```powershell
python --version
py -3 --version
```

The `py` launcher is **optional**. It may be missing while Python itself is installed and working.

### “Python was not found” / Microsoft Store shortcut

First check in a new shell:

```powershell
Get-Command python -ErrorAction SilentlyContinue
where.exe python
```

If necessary, bypass the Store alias and PATH by using Python's full location. Replace `USERNAME` and version with your actual values:

```powershell
& "C:\Users\USERNAME\AppData\Local\Programs\Python\Python314\python.exe" --version
```

- **`&` is PowerShell's call operator**, required before a quoted executable path.
- `Python314` is the typical Python 3.14 folder; Python 3.13 might use `Python313`.
- This is an **example path**, not a universal location.
- Check Windows **Settings → Apps → Advanced app settings → App execution aliases** if `python` launches the Microsoft Store; turn off conflicting Store Python aliases when appropriate.
- Alternatively, add the real Python installation directory to your **user PATH**, then restart PowerShell. No system PATH modification is necessary.

## 4. Convert an OAR to GLB with Web_3d

Open PowerShell in the **Web_3d `dist` folder**, containing `oar2glb.py`. Adapt these example paths:

```powershell
$python = "C:\Users\USERNAME\AppData\Local\Programs\Python\Python314\python.exe"
$oar = "C:\Web3D\MyRegion.oar"
$convoar = "C:\Web3D\Convoar\Convoar\convoar.exe"

& $python .\oar2glb.py $oar --convoar $convoar
```

If `python --version` works, use `python` in place of `& $python`.

**The `oar2glb.py` script belongs to Web_3d**, not Convoar. It runs Convoar, performs repairs and packaging, and normally outputs a `.glb` **next to the source OAR**. Convoar alone outputs glTF files depending on its settings.

### JPEG2000 texture decoding errors

Messages like `Decoded image with unhandled number of components: 0` may appear on some OARs. Check whether conversion actually finishes and a GLB exists before treating a message as fatal.

If texture conversion fails, install Pillow using the **same Python interpreter**:

```powershell
& $python -m pip install Pillow
```

Retry with:

```powershell
& $python .\oar2glb.py $oar --convoar $convoar --reencode
```

Reencoding processes JPEG2000 texture data before running Convoar. Undecodable textures may be replaced with neutral placeholders; **inspect the resulting model visually**. Large OARs can consume significant CPU and RAM; keep the terminal open until the process completes. Check `oar2glb.log` if troubleshooting is required.

## 5. Preview and publish with Web_3d

1. Open `dist/glb-viewer.html` in Edge/Chrome.
2. Choose **Open .glb** and load the converted file.
3. Inspect scene quality and set up the arrival spawn/orientation.
4. For hosting, select the **public export profile** and **Export modular structure (.zip)**.
5. Extract to an HTTP(S) web directory. Expect `index.html`, `NAME.glb`, `NAME.config.json`, `NAME.zones.json`, and `lib/`.
6. To select the region at the site root, create `default.json` containing `{"region":"NAME"}` (without the `.glb` suffix). This is **not** the same as the `default` spawn zone in the zones JSON.

The modular site requires HTTP(S), not opening `index.html` from the file system. Multiple visitors and chat additionally require Web_3d's optional WebSocket presence server.

## 6. Troubleshooting

| Symptom | Fix |
| --- | --- |
| `convoar.exe` not recognized | Run `.\convoar.exe --help` within its folder |
| Missing DLL | Extract **all** ZIP contents and keep DLLs beside the EXE |
| `py` missing | Try `python --version`; `py` is optional |
| `python` opens Microsoft Store | Fix alias/PATH or use full quoted path with `&` |
| `No module named PIL` | Run `& $python -m pip install Pillow` |
| JPEG2000 error | Retry with `--reencode` after installing Pillow |
| Web page cannot locate the region | Check `default.json` and filename consistency |
| Slow scene | Check GLB size; consult Web_3d's tiled-region tools |

## 7. Scope and credits

This ZIP makes it easier to test **OpenSimulator OAR → Convoar → Web_3d GLB**. CI checks that the executable launches on Windows but **does not test full OAR conversion for every release**.

Convoar is © its contributors, Apache-2.0 (see `LICENSE` and `THIRDPARTYLICENSES`). Web_3d is a separate project by [Klaatou](https://github.com/Klaatou/Web_3d) under its own license. Preserve third-party notices for bundled DLLs and ensure you hold rights to publish any OAR assets.
