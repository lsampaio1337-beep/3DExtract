# Guide: Extracting Sprites and Assets from PokemonBR (OTClient)

This guide provides a comprehensive list of methods to extract 2D sprites, animations, and other assets from the PokemonBR client. The client uses a modified version of OTClient (likely OTClientV8 or a similar fork) and protects its assets using `ENC3` encryption within its `data.zip` and `.dat`/`.spr` equivalents.

Since standard extraction tools cannot directly read `ENC3` encrypted files without the specific decryption key (which is embedded inside the `.exe`), you will need to bypass the encryption. You can do this by either dumping the assets directly from memory while the game is running, or by finding the key/using specialized community tools.

Here are the most effective methods to obtain the sprites, ranging from easiest to most advanced.

---

## 1. Texture Dumping via Graphics API (Recommended & Easiest)

Since the game must decrypt the sprites to render them on your screen, you can intercept the unencrypted textures directly from your graphics card memory using graphics debugging tools. PokemonBR provides both OpenGL (`PokemonBR OpenGL.exe`) and DirectX 9 (`PokemonBR Dx9.exe`) executables, which makes this highly viable.

### Option A: NinjaRipper
NinjaRipper is designed specifically to rip textures and models from running DirectX/OpenGL games.
1. Download **NinjaRipper** (v1.7.1 or v2).
2. Open NinjaRipper and set the **Target executable** to `PokemonBR Dx9.exe`.
3. Select the appropriate **Wrapper** (usually `D3D9` or `Intruder`).
4. Set an output directory for the rips.
5. Launch the game through NinjaRipper. Log in and walk around to load the sprites.
6. Press the assigned Hotkey (e.g., `F10`) to rip all currently loaded textures to your output folder.
7. The output will be `.dds` or `.png` files containing the sprite sheets and individual sprites.

### Option B: RenderDoc
RenderDoc is a graphics debugger that captures the entire rendering pipeline for a single frame.
1. Download and install **RenderDoc**.
2. Launch RenderDoc and go to **Tools -> Launch Application**.
3. Point the Executable path to `PokemonBR OpenGL.exe`.
4. Launch the game. You will see an overlay indicating RenderDoc is active.
5. Log into the game. When the sprites you want are visible on the screen, press `F12` (or the capture button in RenderDoc) to capture the frame.
6. Open the capture in RenderDoc, go to the **Texture Viewer** tab, and you will see every single unencrypted sprite and UI element loaded in that frame. You can right-click and save them as PNGs.

---

## 2. Using Specialized OTClient Decryption Tools

The Tibia/OTClient community has developed tools to deal with custom client protections. Since `ENC3` is a known signature used by some OTClient forks, specific tools might be able to decrypt it if it uses default keys or common algorithms.

### Option A: OTClient Sprite Extractor / Unpacker
1. Search GitHub or community forums (like OTLand) for `OTClientV8 unpacker` or `ENC3 decrypter`.
2. Some users have published Python or C++ scripts capable of reading `data.zip` and decrypting `ENC3` if the encryption key matches known defaults used by popular forks (e.g., Mehah's OTClient).

### Option B: ObjectBuilder (Modded)
1. **ObjectBuilder** is the standard tool for opening Tibia `.dat` and `.spr` files.
2. While the vanilla ObjectBuilder cannot open `ENC3` or `data.zip` files directly, some modded versions found on OTLand have added support for OTClient `.otfi` and custom sprites.

---

## 3. Memory Dumping (Process RAM)

If the textures are assembled dynamically and you want the raw `.spr`/`.dat` files, you can dump the game's RAM. The game decrypts these files and loads them into memory when it launches.

### Option A: Process Hacker / Task Manager Dump
1. Launch `PokemonBR.exe` and let it sit on the login screen.
2. Open **Process Hacker** (or Task Manager).
3. Right-click the `PokemonBR.exe` process and select **Create Dump File**.
4. Use a hex editor (like HxD) or a file carving tool (like `binwalk` or `foremost`) on the `.dmp` file.
5. Search the hex dump for known Tibia Sprite file headers or simply use a tool that extracts PNGs/BMPs from binary blobs.

---

## 4. Reverse Engineering the Executable (Advanced)

If you want to create a permanent, automatic extractor for all files in `data.zip`, you need the AES/XOR key used by the `ENC3` algorithm. The key is hardcoded inside the `.exe`.

1. **Static Analysis:**
   - Open `PokemonBR OpenGL.exe` in **Ghidra** or **IDA Pro**.
   - Search for the string `"ENC3"`.
   - Find cross-references to where this string is used. This will lead you to the `decrypt` function.
   - Analyze the function to find the hardcoded key (usually a 16, 24, or 32-byte string/array) and the algorithm (often AES-128/256 CBC or a custom XOR loop).

2. **Dynamic Analysis:**
   - Open the game in **x64dbg** or **Cheat Engine**.
   - Search memory for the string `ENC3`.
   - Set a hardware breakpoint on read/access for that memory address.
   - When the game loads a file (like `Tibia.otfi`), the debugger will pause at the decryption function.
   - You can step through the assembly to watch the game decrypt the buffer in real-time, allowing you to simply copy the unencrypted data out of the memory viewer.
   - Alternatively, you can read the registers passing the key to standard Windows CryptoAPI or OpenSSL functions.

Once you have the key and algorithm, you can write a simple Python script to iterate through every file inside `data.zip`, strip the `ENC3` header, decrypt the payload, and save the raw PNG/OTFI/SPR files to your disk.
