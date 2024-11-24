# Compilation of Commands for Generating, Using, and Managing SSH Keys with FIDO Keys

---

## **1. Generating SSH Keys Using a FIDO Key ([ssh-keygen manual](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html))**

Basic commands to generate SSH keys using a FIDO2 hardware key.

### **(1) Generate a Standard Key**
```bash
ssh-keygen -t ed25519-sk
```
- `-t ed25519-sk`: Generates an ED25519 key using a FIDO2 key.
- During generation, you may be prompted to touch the FIDO key or enter a PIN.

### **(2) Generate a Resident Key**
Store the key on the FIDO2 device itself so it can be used anywhere:
```bash
ssh-keygen -t ed25519-sk -O resident
```
- `-O resident`: Stores the key on the FIDO2 device.
- No private key file is needed on the client PC.

---

## **2. Using SSH Keys Created with a FIDO Key**

### **(1) Add the Generated Public Key to the Server**

1. View the public key:
   ```bash
   cat ~/.ssh/id_ed25519_sk.pub
   ```
2. Add the public key to the server:
   ```bash
   ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub user@server
   ```
   Or manually add it:
   ```bash
   cat ~/.ssh/id_ed25519_sk.pub | ssh user@server "cat >> ~/.ssh/authorized_keys"
   ```

### **(2) Connect via SSH**
```bash
ssh user@server
```
- You will need to touch the FIDO key or enter a PIN when connecting.

---

## **3. Using SSH with a FIDO Key on Another PC**

FIDO2 resident keys can be used on other PCs as well.

### **(1) Initialize SSH Agent and Load the Key**

1. Start the SSH agent:
   ```bash
   eval "$(ssh-agent -s)"
   ```
2. Load the resident key:
   ```bash
   ssh-add -K
   ```
   - `-K`: Adds the resident key stored on the FIDO2 device to the agent.
3. Verify the key loaded in the agent:
   ```bash
   ssh-add -L
   ```
   - If the public key generated with the FIDO2 key is displayed, it has been loaded successfully.

### **(2) Add the Public Key to the Server**

To add the resident key's public key to another server:

1. View the public key:
   ```bash
   ssh-add -L
   ```
2. Add the public key to the server:
   ```bash
   ssh user@server "echo 'sk-ssh-ed25519@openssh.com AAAAB3...' >> ~/.ssh/authorized_keys"
   ```

---

## **4. SSH Key Management Commands**

### **(1) SSH Agent Commands**

- Start the agent:
  ```bash
  eval "$(ssh-agent -s)"
  ```
- Add a key to the agent:
  ```bash
  ssh-add -K
  ```
- List keys loaded in the agent:
  ```bash
  ssh-add -L
  ```
- Delete a key from the agent:
  ```bash
  ssh-add -d <key>
  ```

---

### **(2) SSH Key Generation Commands**

- Generate a standard SSH key:
  ```bash
  ssh-keygen -t ed25519
  ```
- Generate an SSH key using a FIDO2 key:
  ```bash
  ssh-keygen -t ed25519-sk
  ```
- Generate a resident key:
  ```bash
  ssh-keygen -t ed25519-sk -O resident
  ```
- Specify scope and UserID:
  ```bash
  ssh-keygen -t ed25519-sk -O resident -O application={scope:}{userid}
  ```
- Save the SSH key in PEM format:
  ```bash
  ssh-keygen -m PEM
  ```

---

## **5. Other Management Commands**

### **(1) Copy Public Key**

View the public key and copy it to the server:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub user@server
```

### **(2) Retrieve Public Key from FIDO Key**

Load the public key from the resident key:
```bash
ssh-add -K
ssh-add -L
```

### **(3) Configure SSH Keys on the Server**

1. Check `authorized_keys` on the server:
   ```bash
   cat ~/.ssh/authorized_keys
   ```
2. Set permissions:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

### **(4) Connecting USB Devices to WSL2 ([Microsoft Docs](https://learn.microsoft.com/en-us/windows/wsl/connect-usb))**

To load the public key from the resident key in WSL2:
```bash
winget install --interactive --exact dorssel.usbipd-win
-------------------------------------------------------
usbipd list
usbipd bind --busid <busid>
usbipd attach --wsl --busid <busid>
usbipd detach --busid <busid>
```

---

## **6. `fido2-token` Commands ([fido2-token Manual](https://developers.yubico.com/libfido2/Manuals/fido2-token.html))**

### **(1) View List of Connected Authenticators**
```bash
$ fido2-token -L
/dev/hidraw0: vendor=0x20a0, product=0x42b1
```

### **(2) View Resident Credentials on the Device**
```bash
$ fido2-token -L -r /dev/hidraw0
Enter PIN for /dev/hidraw0:
00: 4wYQ6KFiEVl...eXDIcivHisb8= ssh:
01: W8qXHGsr2Mz...rBoR9vGXmhvw= ssh:rpi5-1
```

---