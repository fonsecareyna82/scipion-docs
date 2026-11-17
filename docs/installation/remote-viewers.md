# Optional: Desktop Viewers in the Browser

Some results are best seen in desktop programs such as ChimeraX or IMOD's 3dmod. When users work from another computer, ScipionWeb can run these viewers **on the server, next to the data**, and show their window **in a floating window inside the web page**.

- **No data moves over the network.** The program opens the files where they are; only the picture of its window travels to the browser.
- **Users install nothing.** A browser is enough.
- **It is optional.** Without it, ScipionWeb works as before: viewers open on the screen of the machine that runs the server.

This uses [Xpra](https://xpra.org/), a free program that gives each viewer its own invisible screen on the server and shows that screen in a web page.

---

## How it behaves

| Where the browser is | What happens when a viewer is opened |
|---|---|
| On another computer | The viewer runs on the server and appears in a floating window inside ScipionWeb |
| On the server itself, in front of its screen | The viewer opens on that screen, as before |
| Any, but Xpra is not installed | The viewer opens on the server's screen, as before |

- Closing the floating window closes the program on the server, and closing the program closes the window.
- After reloading the page, the viewers that were open come back.
- Each user can have **3 viewers** open at the same time.
- A viewer that nobody uses for **15 minutes** is closed.

---

## 1. Install Xpra (Ubuntu)

This needs administrator rights (`sudo`). It adds the official Xpra repository and installs Xpra and its web page:

```bash
# Ubuntu version name: jammy (22.04), noble (24.04)…
CODENAME=$(lsb_release -cs); echo $CODENAME

sudo wget -O /usr/share/keyrings/xpra.asc https://xpra.org/xpra.asc
sudo wget -O /etc/apt/sources.list.d/xpra.sources \
  https://raw.githubusercontent.com/Xpra-org/xpra/master/packaging/repos/$CODENAME/xpra.sources

sudo apt update
sudo apt install -y xpra xpra-html5
```

Check it:

```bash
xpra --version
```

For other systems (Rocky, AlmaLinux, Fedora, Debian…), follow the instructions on the [Xpra download page](https://github.com/Xpra-org/xpra/wiki/Download). Install both `xpra` and `xpra-html5`.

!!! note "Use the official repository"
    The `xpra` package of the distribution itself is often old and may come without the web page (`xpra-html5`). Install it from the Xpra repository as shown above.

---

## 2. Restart ScipionWeb

```bash
scipionapi restart
```

Then reload ScipionWeb in the browser. Nothing else needs configuring.

---

## 3. Check it works

From **another computer**:

1. Open a project and select a protocol with a volume output.
2. Open the output with its desktop viewer.
3. A floating window should appear with "Opening the viewer on the server…", and then the program.

If no floating window appears (the viewer opens on the server's own screen, or fails to open), ScipionWeb did not find Xpra: check `xpra --version` as the user that runs ScipionWeb and restart it again.

---

## Network and security

- **No new ports.** Everything goes through the same address and port as ScipionWeb. Xpra only listens inside the server.
- Each viewer has its own random key and password. A user only sees and closes their own viewers.
- File transfer, printing, sound, webcam and starting other programs from the viewer's page are turned off.
- Behind a reverse proxy (nginx, Apache), the proxy must pass **WebSocket** connections to ScipionWeb under `/api/`.

---

## Several machines

The viewers run on the machine that runs the ScipionWeb server (the main node). Install Xpra there. The data must be readable from that machine at the same paths, as is already required for multi-node setups.

---

## If 3D is slow

By default the picture is drawn by the server's processor. This is fine for most maps, but large maps or complex scenes may feel slow. If the server has a GPU, [VirtualGL](https://virtualgl.org/) lets the viewers use it. It is not needed to start.
