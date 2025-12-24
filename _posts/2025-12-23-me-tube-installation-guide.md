---
layout: post
draft: false
title: "MeTube Installation Guide"
date: 2025-12-23
description: "This guide documents how to install and setup MeTube after fully evaluating the more popular alternative, Tube Archivist.  MeTube downloads YouTube videos to my network storage for streaming later in a minimized app (phone or pc) so just the audio portion is used." 
---


## MeTube Installation for YouTube Video Downloads

This guide documents the installation and configuration of MeTube as a web-based YouTube downloader on VM 1106 (samba-storage-vm). MeTube provides a simple web interface for yt-dlp, allowing easy video downloads from any device on the network.

### Background and Tool Evaluation

Before settling on MeTube, we evaluated Tube Archivist as a potential solution. While Tube Archivist offers powerful features like Elasticsearch-based search and metadata indexing, it was rejected for this use case because:

- Designed for archiving entire YouTube channels automatically
- Queues all videos from subscribed channels by default
- Requires manual ignoring of unwanted videos
- Uses significant resources (Elasticsearch, Redis, 4 GB+ RAM requirement)
- Overly complex for downloading individual videos on-demand

MeTube was chosen because it provides exactly what was needed: a simple web interface for downloading specific videos with full manual control.

### System Information

- Host VM: samba-storage-vm  
- IP Address: 192.168.1.xxx
- Tailscale IP: xxx.xxx.xxx.xxx
- Download Location: /mnt/storage/YouTube-downloads
- Port: 8081

### Prerequisites

- Docker and Docker Compose installed on VM 
- Network storage already configured and mounted at /mnt/storage
- Tailscale installed and configured on VM

### Installation Steps

Navigate to the docker directory and create the MeTube folder:

```sh
mkdir -p /home/{{username}}/docker/metube
cd /home/{{username}}/docker/metube
```

Create the docker-compose.yml configuration file:

```sh
nano docker-compose.yml
```

Add the following configuration:

```yaml
services:
  metube:
    image: ghcr.io/alexta69/metube
    container_name: metube
    restart: unless-stopped
    ports:
      - "8081:8081"
    volumes:
      - /mnt/storage/YouTube-downloads:/downloads
    environment:
      - UID=1000
      - GID=1000
      - DOWNLOAD_DIR=/downloads
      - OUTPUT_TEMPLATE=%(uploader)s/%(title)s.%(ext)s
      - YTDL_OPTIONS={"writesubtitles":true,"writeautomaticsub":true,"subtitleslangs":["en"],"postprocessors":[{"key":"FFmpegSubtitlesConvertor","format":"srt"}]}
```

Save the file and exit nano.

Start the MeTube container:

```sh
docker compose up -d
```

Verify the container is running:

```sh
docker ps
```

The output should show the metube container with status "Up".

### Configuration Details

The docker-compose configuration includes:

- Download directory mapped to network storage at /mnt/storage/YouTube-downloads
- UID/GID set to 1000 to match the {username} user
- OUTPUT_TEMPLATE organizes videos by channel name automatically
- YTDL_OPTIONS configured to download English subtitles and convert them to SRT format

### Accessing MeTube

MeTube can be accessed via two URLs:

Local network access:

```
http://192.168.1.167:8081
```

Tailscale access (works from anywhere):

```
http://100.76.118.67:8081
```

Using the Tailscale IP address is recommended for simplicity, as it works both at home and remotely.

### Using MeTube

Open MeTube in a web browser using one of the URLs above.

Configure the download options:
- Format: Select "mp4" for maximum compatibility
- Quality: Select "Best" to download the highest available quality
- Auto Start: Leave as "Yes" for immediate downloads
- Download Folder: Leave as "Default" or create custom folders for organization

Paste a YouTube video URL into the input field and click Add.

The video will begin downloading immediately. Download progress is shown in real-time.

Completed downloads appear in the "Completed" section and are automatically organized into channel folders at /mnt/storage/YouTube-downloads.

### File Organization

MeTube organizes downloaded videos using the channel name (uploader) as the folder structure:

```
/mnt/storage/YouTube-downloads/
├── Channel_Name_1/
│   ├── video1.mp4
│   ├── video1.en.srt
│   ├── video2.mp4
│   └── video2.en.srt
└── Channel_Name_2/
    ├── video1.mp4
    └── video1.en.srt
```

Note that MeTube may use spaces in channel names while the yt-dlp command-line alias uses underscores. Both are correct and represent the same channels.

### Integration with Infuse

Videos downloaded via MeTube are immediately accessible in Infuse on iOS devices:

Access the network storage via SMB in Infuse using the Tailscale IP:

```
smb://100.76.118.67/storage
```

Navigate to the YouTube-downloads folder to see all downloaded videos organized by channel.

Videos can be played directly in Infuse with full support for:
- Background audio playback
- Picture-in-picture mode
- Casting to TV via AirPlay or Chromecast
- Subtitle display

### Remote Access via Tailscale

With Tailscale installed on both VM 1106 and mobile devices, MeTube can be accessed from anywhere:

On iPhone or other mobile device, open Safari or preferred browser.

Navigate to the Tailscale IP address:

```
http://100.76.118.67:8081
```

The full MeTube interface is available, allowing video downloads from any location.

Downloaded videos are saved to network storage and immediately accessible via Infuse.

### Complete Workflow

The complete YouTube download and viewing workflow:

Browse YouTube using FreeTube (desktop) or regular YouTube app to find desired videos.

Copy the video URL.

Access MeTube via Tailscale IP on phone or laptop.

Paste the URL, select mp4 format, and click Add.

Video downloads to network storage organized by channel name.

Open Infuse on iPhone to access the video immediately.

Watch directly in Infuse or cast to TV with full playback controls.

### Troubleshooting

If MeTube is not accessible, verify the container is running:

```sh
docker ps
```

Check container logs for errors:

```sh
docker compose logs -f
```

If downloads are not appearing in the expected location, verify the mount point:

```sh
ls -la /mnt/storage/YouTube-downloads
```

Ensure the directory has proper permissions for the {username} user (UID 1000).

If Tailscale access fails, verify Tailscale is running on VM 1106:

```sh
tailscale status
```

Restart the container if needed:

```sh
docker compose restart
```

### Updating MeTube

MeTube includes automatic yt-dlp updates in nightly builds. To update MeTube to the latest version:

```sh
cd /home/{username}/docker/metube
docker compose pull
docker compose up -d
```

This pulls the latest image and recreates the container with the new version.

### Comparison with Command-Line yt-dlp

The existing yt-dlp command-line alias remains available and can be used alongside MeTube:

```sh
yt-dlp-save "https://youtube.com/watch?v=VIDEO_ID"
```

Both methods download to the same location and organize files similarly. MeTube offers advantages for:
- Mobile device access without terminal
- Visual progress tracking
- Queue management for multiple downloads
- Web-based interface accessible from any device

The yt-dlp alias remains useful for:
- Quick downloads from laptop terminal
- Scripting and automation
- Advanced yt-dlp options not exposed in MeTube UI

### Additional Notes

MeTube supports downloading from over 1000 video sites beyond YouTube, not just YouTube content.

Downloaded files are immediately accessible to any device connected to the network storage.

The subtitle configuration automatically downloads English subtitles when available and converts them to SRT format for maximum compatibility.

Videos downloaded via MeTube integrate seamlessly with media center applications like Jellyfin or Plex if needed in the future.
