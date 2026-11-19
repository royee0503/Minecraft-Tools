# Minecraft Tools

[繁體中文](README.md) | [English](README_EN.md)

### Features

This project contains two independent tools:

- **Mod downloader**: Downloads mods for a specified Minecraft version from Modrinth, based on the mod list defined in `mods.json`.
- **Resource pack downloader**: Generates and downloads resource packs for a specified Minecraft version from Vanilla Tweaks. The pack selection matches TaurusTweaks, and the matching list is chosen automatically by version (26.x and later, or 1.21 and earlier).

### Requirements and Usage

- Python 3.9 or later
- Use Python's built-in `venv` module to create an isolated virtual environment.

### Set Up a Virtual Environment (Windows PowerShell)

#### 1. Navigate to the project directory
```powershell
cd .\Minecraft-Tools
```

#### 2. Create an isolated virtual environment
```powershell
python -m venv .venv
```

This creates a `.venv` directory inside the project:
```text
Minecraft-Tools/
├── .venv/
├── modrinth_downloader.py
├── vanillatweaks_downloader.py
└── ...
```

#### 3. Activate the virtual environment
```powershell
.\.venv\Scripts\Activate.ps1
```

After activation, `(.venv)` appears at the beginning of the PowerShell prompt, for example:
```text
(.venv) PS C:\Users\<username>\Minecraft-Tools>
```

#### 4. Verify that Python is running from the virtual environment
```powershell
python --version
where.exe python
```

The first Python path shown by `where.exe python` should be inside the project's `.venv\Scripts\` directory.

#### 5. Install the project dependency
```powershell
python -m pip install requests
```

#### 6. Run the programs
```powershell
python modrinth_downloader.py
python vanillatweaks_downloader.py
```

To leave the virtual environment, run `deactivate`. When you open a new terminal later, navigate to the project directory and run the activation command again.
Follow the on-screen prompt to enter a Minecraft version. Both programs can be cancelled at any time: for the mod downloader, enter `q` or press Ctrl+C; for the resource pack downloader, press Q (it cancels immediately in a Windows terminal; elsewhere, enter `q` and press Enter).

### Safety Principles

- Both programs only add new files. They never overwrite or delete your existing files (they only clean up their own `.part` temporary files).
- Mod downloader: Verifies file size and hash after downloading (SHA-512 preferred, SHA-1 as a fallback). Files without a hash are not downloaded, and files that fail verification are not retained. If a file with the same name already exists, it is skipped. When a mod has a newer version and the old jar is still in the folder, the program only reminds you to remove it manually and never deletes it automatically.
- Resource pack downloader: Downloads to a temporary file first, and only saves it as the final file once it is confirmed to be a valid ZIP. If the file name is taken, it adds (1), (2), and so on. After downloading, it reads `Selected Packs.txt` inside the ZIP to check that every requested pack was included.

### Code Structure (Read from Top to Bottom)

#### `modrinth_downloader.py`

1. Configuration and Constants
2. Data Structures (`Config` / `ModResult`)
3. Load Configuration File
4. Modrinth API
5. Select Versions and Files
6. Download and Verification
7. Installed Mods List (`installed.json`)
8. Process Individual Mods
9. Report
10. Main Program

#### `vanillatweaks_downloader.py`

1. Basic Settings (including the two resource pack lists)
2. Custom Error Classes
3. Enter and Check the Version
4. Get Category Data
5. Ask the Site to Generate the ZIP
6. Download the ZIP
7. Check the ZIP Contents
8. Main Program
9. Program Entry Point

### File Descriptions

#### `modrinth_downloader.py` - The Minecraft Mod Downloader (Modrinth)

The main program for the automatic mod downloader. You do not need to change the program itself; just edit `mods.json`.

- When it runs, it asks for the target Minecraft version and checks your input against Modrinth's version list.
- For each mod, it picks the newest suitable version according to the loaders and version types in `mods.json`. If none is found, it explains why (for example, the mod has not been updated yet, has no build for that loader, or only has beta releases). This is not an error, and you can simply run it again later.
- Files are downloaded to `<repository_dir>/<Minecraft version>/`.
- `installed.json` records the file name currently downloaded for each mod and is used to detect old jars; `download_report.json` is the full report of the run.
- If a mod has required dependencies, the program only reminds you to check them and does not download them automatically.

#### `mods.json`

You can edit this file to specify which mods you want to download automatically. Example:

```json
{
  "repository_dir": "~/Downloads/Minecraft Mod Repository",
  "loaders": ["fabric"],
  "version_types": ["release"],
  "mods": [
    { "name": "Apple Skin", "project": "appleskin" },
    { "name": "Architectury", "project": "architectury-api" }
  ]
}
```

Required dependencies are not downloaded automatically, so library mods such as Fabric API, Architectury, and Cloth Config should be listed in the file as well (as in the example above).

- `mods` (required): The list of mods. `project` is the Modrinth project slug or project ID; `name` is an optional display name.
- `loaders` (optional, default `["fabric"]`): Which loaders to download versions for.
- `version_types` (optional, default `["release"]`): Accepted version types: `release`, `beta`, or `alpha`.
- `repository_dir` (optional, default `~/Documents/Minecraft Mod Repository`): Where files are saved. A folder is created for each Minecraft version inside it.

#### `vanillatweaks_downloader.py` - The Minecraft Resource Pack Downloader (Vanilla Tweaks)

The main program for automatically downloading resource packs. It asks the Vanilla Tweaks website to generate the specified resource packs and downloads them.

- On startup it reads the versions currently offered on the website and lists them, so new versions appear without changing the program.
- Version input: leave it empty and press Enter to use the latest version; press Q to cancel. Invalid input, or a version the website does not have, asks you to enter it again.
- Minor versions are mapped to their major version automatically: 1.21.1 to 1.21.11 all use 1.21, and 26.3.1 uses 26.3.
- The pack list is chosen by version: 26.x and later use `PACKS_26_3` (125 packs), and 1.21 and earlier use `PACKS_1_21_11` (122 packs). The two lists come from `Selected Packs.txt` inside the TaurusTweaks 26.3 and 1.21.11 resource pack ZIPs.
- Packs the website does not yet have for a given version are reported as "not updated yet" and are not treated as errors.
- The file is saved as `~/Downloads/VanillaTweaks_MC<the version you entered>.zip`.
- To add or remove packs, edit the lists in the file (use the website's internal names, such as `NoFog`).

### Notes

- This is a personal, unofficial tool and is not affiliated with Modrinth or Vanilla Tweaks.
- The resource pack downloader imitates a browser using the Vanilla Tweaks website, so it may stop working if the site changes. Please do not run it in loops or on a schedule, to avoid putting load on the site.
- What you get is a ZIP generated by the Vanilla Tweaks website. The pack selection is the same, but it does not include the custom name and icon set by the TaurusTweaks author.

---

The original code and documentation written for this project are licensed under the **GNU General Public License v3.0 only (GPL-3.0-only)**. See the [LICENSE](LICENSE) file in the repository root for the full license text.

You may use, study, modify, and distribute this project. If you distribute this project or a derivative work, you must comply with GPL-3.0-only, including providing the corresponding source code and licensing GPL-covered derivative works under the same license.

### Third-Party Software and External Content

This project does not relicense third-party packages, external website content, or other authors' works under GPL-3.0-only. They remain subject to the licenses and terms provided by their respective authors, maintainers, or sources.

- **Python packages and dependencies**: Subject to the licenses of the specific packages and components used.
- **Modrinth**: Mod files and project content remain subject to the licenses specified by their respective authors.
- **Vanilla Tweaks**: Generated resource packs and related assets remain subject to their original licenses and the website's terms of use.
- **TaurusTweaks**: This project references the `Selected Packs.txt` files contained in its ZIP archives to compile resource-pack selection lists; those lists and related content are not relicensed by this project.

When distributing this project, separately verify the license requirements for any third-party components, mods, resource packs, or data that are included or downloaded.
