# Mlx, X11, XServer → Texture Raycasting & Rendering
***
![Logo](Img/MLX.png)
***

***Note:** Preplexity AI was used in information gathering & learning the concepts and writing the components of this article, the information presented here might be imprecise or incorrect, its always on you to dive deeper to find real and accurate knowledge.*

# ***X11 Protocol:***

X11 is a network-transparent protocol and windowing system that powers graphical user interfaces on Unix-like OS, separating applications (clients) from display management (server).

Clients send requests over a socket connection to draw windows, handle input like mouse/keyboard events, and receive replies, which controls hardware rendering and input routing.

---

# ***Xserver:***

Xserver, is the one responsible for drawing windows graphically on the displaying screen, its also responsible for handling and managing the mouse/keyboard.

We call it a server, because it can not only display the windows on the local monitor only, but on different systems on the network that also runs ***Xserver*** on their software.

Using the Display environment variable to control where to display our output. The display environment variable contains the IP Address of the system where is going to send its display output

Until 2004, Xfree86 was the default version of X server that was used by most linux distributions, it works and functions pretty much as the modern one which is X.org-X11, which is the one used in the present modern systems. (X.org-X11 is based on Xfree86).

- **Window Manager :** Xserver creates the window in the GUI, but the window manager manages and customize how the window looks like and how they behave
- **Desktop Environment:** it leverages the look and feel created by the window manager and then adds a series of tools and utilities to make the GUI more useful, as it ties all the graphical user interface components together to make it easier, some security systems does not implement the desktop environment.
- There are programs that allows you using SSH & X11 forwarding, to open and render the desktop environment of other systems on your local systems as long as they are on the same networks (Software such as puTTY).

**Resources**  [[Linux Remote Access | SSH and X11 Forwarding](http://youtube.com/watch?v=auePeI8vZA8)] | [[X Window System](https://www.youtube.com/watch?v=mV1TNyWGQQ8)].

---

# ***MLX:***

MiniLibX is a simplified graphics library used in projects from 42 Schools, acting as a thin wrapper around the X11 protocol for basic window creation, pixel drawing and event handling on Linux.

It connects to an Xserver by opening a socket on port 6000, this port is traditionally used by the X window System (X11) as the default port for the primary display. Each X11 display listens on port 6000 plus the display number, so Display:0 uses port 6000, Display:1 listens on port 6001 (6000 + Display Number).

The connection gets initialized via Xlib functions like XOpenDisplay and uses XCreateSImpleWindow to set up a window with event masks for inputs like KeyPress or Expose.

- ***MLX & X11 Flow:***

 MLX initializes by calling XOpenDisplay → X11 Connection established → Window creation using XCreateWindow or XCreateSimpleWindow → setting up an image context via XCreateIMage for pixel buffers → mlx_loop (Wrapping XNextEvent) to dispatch hooks for keyboard, mouse…

- ***Authorization & Xauthority:***

When an X server starts (Xorg on:0), it generates a 128-bit MIT-MAGIC-COOKIE-1 key, 

a hex string, and adds an entry to the server’s .Xauthority file, when Client invokes XOpenDisplay (as MLX does internally via Xlib), which reads the user’s Xauthority file, locates the target display, and sends the cookies in plain text during connection setup, the server compares it against its own copy, granting access only on exact match.

**Resources** : [X.org - X11 Documentation](https://www.x.org/releases/X11R7.7/doc/xproto/x11protocol.html)

---

# *Rendering Textures:*