# 31C3 - Webseite/Player

The code for the 31C3 Player-Webseite is located at GitHub: https://github.com/voc/frontends/tree/master/31c3

It runs on **live.ber.c3voc.de** in **/srv/nginx/31c3-frontend/31c3** which is the master server of a cascade of caching servers which will be served via the DNS-Name **streaming.media.ccc.de**. This is also the public released URL: http://streaming.media.ccc.de/

The Frontend is implemented in PHP without any special frameworks or libraries on the server side. There is a rewrite-Rule configured in the nginx.conf (the .htaccess is just for apache development) which redirects all Requests towards the [index.php](index.php). Within this file the URL is analyzed and routed to one of the following Pagetypes:

### Pagetypes
 - [rooms.php](pages/rooms.php) - startpage, list of all rooms, minirooms and party streams
 - [room.php](pages/room.php) - a playerpage for a normal or a miniroom
 - [party.php](pages/party.php) - a playerpage for one of the party/music streams
 - [about.php](pages/about.php) - the "about this" page
 - [program-json.php](pages/program-json.php) - generating a special fahrplan json with more timing information which is used to show the current and upcoming talks on the main page
 - [404.php](pages/404.php) - a cute 404 page for those cases, when someone is not able to copy a link properly

despite that there's [500.html](500.html) which is a dependency-free single-file absolutely bare minimal version of a styled error page that can be served by the caching servers when an error condition occurs (ie. upstream server unavailable).

### Minirooms
A miniroom is a room without some features. Currently the only planned miniroom is the "Sendezentrum". Minirooms are like normal rooms wit the following differences:

 - no live audio-translation
 - *(this list was longer once)*

### URL Building
The other direction (building urls from parameters) is done using small helper functions in [lib/helper.php](lib/helper.php). Obviously the URLs built there needs to match the Regeses in index.php mapping them back to parameters.

### Templates
The Pagetype-Scripts do general logic. I guess I should have called them Views. They analyze the URL and Config and pass some calculated parameters into the Templates. All Pagetypes call into the [page.phtml](template/page.phtml)-Template first which sets up the basic HTML and includes common page parts. It also includes the Content-Template *($page.phtml)* which the Pagetype requested. The page content is then generated in [rooms.phtml](pages/rooms.phtml), [room.phtml](pages/room.phtml), [party.phtml](pages/party.phtml), [about.phtml](pages/about.phtml) and [404.phtml](pages/404.phtml) which follow the same semantics as the Pagetypes.

The templates include smaller chunks that are offten reused in multiple templates. They are called [assemblies](template/assemblies).

### Switcher
The assemblies in the [switcher](template/assemblies/switcher) directory are used to list al lavailable formats as links below the players. They also provide directlinks to the various stream formats and languages.

### Program View
The Program-View is generated by reading the configured fahrplan-json and filling the gaps with virtual events (like *gap*, *pause* or *daychange*). This logic is located in [lib/program.php](lib/program.php). The Program is fetched and parsed on every request. Caching is handled by the cascaded caching Servers mentioned earlier. One of the users of this helper function is the program template-assebly [program.phtml](template/assemblies/program.phtml). which generates the HTML for the complete program section with all talks of the whole Congress. Finding the current Talk by local Time and scrolling the program view to the right place is a task for [javascript](assets/js/lustiges-script.js#L190).

### Startpage Program Teaser
The Startpage Teaser works somewhat different. It is completely controlled by [javascript](assets/js/lustiges-script.js#L304). The Javascript-Code requests the *program.json* which is actually handled by the [program-json.php](pages/program-json.php) Pagetype (and also cached by the cascade). It figures out which talk is currently running and which is upcoming and renders them into the Startpage.

### Subtitles Viewer
Also a complete [JS-Only](assets/js/lustiges-script.js#L34) feature, it adds a Subtitles-Button to the MediaElement.js-Player. On-Click a display-line above the video element is displayed. Socket.io is loaded and a Socket-Connection to the [subtitling-backend](https://github.com/c3subtitles/live-subtitles-test-backend) is opened. Incoming Text is automatically resized to fit the line's height.

### MediaElemet.js & Flash
Our main distribution formats are h264/rtmp, h264/hls and webm/icecast. The beast real-time experience can be obtained by the rtmp-Stream but unfortunately no browser is capable of playing rtmp. HLS is only really [supported on mobile devices](http://www.jwplayer.com/html5/hls/). The WebM Stream would be really good to have but a) is encoding a HD-WebM-Stream in Realtime a task for a really fast single CPU (ffmpeg/WebM is not multithreaded) and b) does our HD-WebM-Stream have some issues with keyframes and GOP and such.

So we need Flash to play in the Browser, although we encourage users to use their VLC or ffplay or whatever instead. The version of MediaElemet.js and especially the swf-Player is exactly the same as it was at the 30C3 (2013), because newer Versions seem to have issues with scaling down the video image and keeping the aspect ratio in fullscreen. Just keeping the *known working* Versions was the simplest thing to do.

### Contact
If you have ideas for changes - especially during the Congress itsself - just open a Pull request. I'll be actively monitoring the Website, this repo and all incoming changes. If yo uwant to chat you can find me (MaZderMind) in #voc at irc.hackint.eu.