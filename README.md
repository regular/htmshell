# htmshell
A Desktop Environment/Graphical Shell for Linux implemented exclusively with web technologies. Desktop extensions/widgets and entire applications can be implemented in HTML and Javascript using any or no web framework, and can be distributed via npm. htmshell (pronounce H.T.M shell) runs on top of [weston][1] and consists of the following components:

- [html-terminal][2] – a terminal emulatar that understands both, [classic VT100/ANSI escape sequences][10] and HTML/CSS/JavaScript (embedded in a non-standard escape sequence)
- [domfs][3] – a bridge between UNIX and the DOM, implemented as a userland filesystem that runs in the browser and exposes the DOM elements as files and directories to the Linux kernel. (if that sounds crazy, that's because it is)
- [remote-spawn][4] – invoke shell commands (parsed by bash) via websockets in a slightly more secure way than what it sounds like!
- A small collection of JavaScript modules that provide desktop widgets
  - [htmshell-mpc][5] A widget for controlling mpd[6], the music player daemon
  
<details><summary>some future plans</summary>
- More Widgets:
  - [htmshell-rgbkey][7] A widget for controlling the per-key RGB backlight of a Razer keyboard
  - [htmshell-networkmanager][8] A widget that lets you choose a wifi network
  - [htmshell-graphs][9] A widget that shows realtime graphs for things like memory consumption, CPU temperature, and battery charge
  
- an Arch-based distro
</details>
  
The project is at an early proof-of-concept stage. (Two days old at the time of writing)

# Motivation

When thinking about which desktop features I actually use in my day-to-day work as a developer and teacher, I realized it pretty much boils down to a terminal emulator and a web browser. Everything I do takes place in either one or the other.

Thinking about it, a browser really is just another kind of terminal emulator. Both are being used to parse and interpret a program's output stream and present it in a human-friendly way, both take user input and provide it to the program. Terminal emulators interpret ANSI sequences and display colorful monospaced characters (the best kind of character) and send keystrokes to programs. Browsers on the other hand interpret html, css, svg and all kinds of binary formats, like png, jpeg, mp3, wav, ogg, pdf, and pretty much all video formats, and they send keyboard, mouse and touch events to programs. On top of that, they can render hardware-accelerated 3D graphics and compile [Open GL Shaders][13]. This feature set is so rich, you barely need anything else to implement a really nice desktop environment. The only thing that is missing is a better integration between the browser and the terminal, or, in more general terms, between the browser and UNIX.

### Browser-backed system terminal

First of all, it would be nice if the browser would also be able to act as a terminal emulator. Turns out, that's an easy one, others already did all the work of implementing a fully functional and fast [terminal emulator in Javascript][11]. That terminal emulator however needs a way to connect to a shell, like bash or zsh. Because the terminal emulator runs in the browser's sandbox, its communication is restricted to http. However, using websockets, we can connect it to a local http server that runs the shell and pipes stdin and stdout back and forth between the shell and the terminal. This is what [html-terminal][2] does. If we want this terminal to replace the Linux system terminal, it should be full-screen and uncluttered. html-terminal provides a minimal, yet fully-featured browser for that porposes – all that is removed is the browser's user interface ([the "chrome"][15]). This no-clutter browser is implemented in [a few lines of C code][14]. It's just an extremely thin wrapper around [webkit2gtk][12] (basically the "hello world" of webkit programming).

On my machine, [systemd][16] boots right into html-terminal and the standard Linux login prompt already is rendered by the browser in my favourite color scheme. From here I can run ssh, tmux and vim, which already covers 50% of what I need.

### A UNIX-style DOM API (or "mad science part I" )

Unix' design is beautiful. The entire system is represented in one unified tree. Your root file system in `/`, devices under `/dev`, running processes under `/proc`. This tree contains more than just files and directories. Simply by writing some data to the right file, you can do things like change the brightness of your screen.

> If you try 'cat /boot/vmlinuz > /dev/dsp' (on a properly configured system) you should hear some sound on the speaker. That's the sound of your kernel! A file sent to /dev/lp0 gets printed. Sending data to and reading from /dev/ttyS0 will allow you to communicate with a device attached there - for instance, your modem.
_from [The Linux Documentation Project][17]

By reading files under `/proc` you can find out [all kinds of things][18] about programs tht are currently running and you can change some of the kernel's behaviour at runtime. Point being, the file system _is an API_. You use it with standard tools like `ls`, `cat`, `echo`, `cp`, etc.

That's the spirit of domfs. Just like `/proc` exposes details about running processes, `/dom` exposes details about the html document currently loaded in your browsers. (It doesn't have to be `/dom` and it can be more than one document). Under this directory, you'll find the DOM elements of an HTML page. You have access to the html they contain (the `innerHTML`) and attribute values like `class`, `id` or `name`.

The beauty of this is that standard tools like `echo` and `cat` can now manipulate the DOM. That means thay can be used to play videos, display a CSS-styled UI or even turn database tables into 3D charts that you can walk through! Oh, the possibilites! You can also set up a couple of UI widgets from the comfort of your `~/.bashrc~` or `~/.zshrc`.

Because your terminal emulator now is capable of displaying html and running Javascript, these widgets show up right on top of your terminal. No more alt-tabbing, the GUI is where you are.

### 

[1]: https://github.com/wayland-project/weston
[2]: https://github.com/regular/html-terminal
[3]: https://github.com/regular/domfs
[4]: https://github.com/regular/remote-spawn
[10]: https://en.wikipedia.org/wiki/ANSI_escape_code
[11]: https://github.com/macton/hterm
[12]: https://directory.fsf.org/wiki/Webkit2gtk
[13]: https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language
[14]: https://github.com/regular/html-terminal/blob/master/web.c
[15]: https://en.wikipedia.org/wiki/Graphical_user_interface#User_interface_and_interaction_design
[16]: https://en.wikipedia.org/wiki/Systemd
[17]: http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/dev.html
[18]: http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/proc.html
