1. First, attempt to install on Windows assuming you have already downloaded anaconda:


   a. Once Anaconda is installed, open Anaconda Prompt and create a new environment:
      ```
      conda create --name pynbody
      ```

   b. Activate the environment:
      ```
      conda activate pynbody
      ```

   c. Install pip:
      ```
      conda install pip
      ```

   d. Install pynbody:
      ```
      pip install pynbody
      ```

   e. Test the installation:
      - Open a Python file or Jupyter notebook (you will have to install Jupyter for the environment)
      - Try importing pynbody:
        ```python
        import pynbody
        ```

2. If the Windows installation fails, try using windows subsytem for linux (WSL):

WSL Installation:

1. Install WSL:
   - Follow the directions at https://learn.microsoft.com/en-us/windows/wsl/install
   - I'd recommend using Ubuntu, which you can get from the Windows Store.

2. Open Ubuntu on WSL:
   - Open the Start menu and search for "Ubuntu"
   - Click on the Ubuntu app to launch the terminal

3. Update and upgrade packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

4. Download Anaconda:
   ```bash
   cd ~
   curl -O https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh
   ```
   Note: Check the [Anaconda website](https://docs.anaconda.com/anaconda/install/linux/) for the latest version and more instructions. 

5. Install Anaconda:
   ```bash
   bash Anaconda3-2024.06-1-Linux-x86_64.sh
   ```
   - Follow the prompts:
     - Press Enter to review the license agreement
     - Type "yes" to accept the license terms
     - Confirm the installation location (default is recommended)
     - Type "yes" when asked if you want the installer to initialize Anaconda3

6. Update your shell:
   ```bash
   source ~/.bashrc
   ```

7. Verify Anaconda installation:
   ```bash
   conda --version
   conda list
   ```

8. Create and activate a new environment:
   ```bash
   conda create --name pynbody
   conda activate pynbody
   ```

9. Install pynbody:
   ```bash
   conda install pip
   pip install pynbody
   ```

10. Test the installation:
    - Open a Python file or Jupyter notebook
    - Try importing pynbody:
      ```python
      import pynbody
      ```
