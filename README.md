# Setting Up Plex Server with Docker

## Prerequisites
1. **Install Docker**: Follow the instructions at [Docker Installation Guide](https://docs.docker.com/get-docker/).
2. **Install Docker Compose**: Follow the instructions at [Docker Compose Installation Guide](https://docs.docker.com/compose/install/).

---

## Step 1: Obtain Your Plex Claim Token
1. Go to [Plex Claim](https://www.plex.tv/claim/) and log in to your Plex account.
2. Copy the claim token provided.

---

## Step 2: Get Your IP Address
1. Open a terminal.
2. Run the following command in bash to get your IP address:
    ```bash
    sudo ip a | grep "inet "
    ```
3. Note the IP address (e.g., `10.0.0.146`).

---

## Step 3: Create Environment Variables File
1. Create a file named `.env` in the same directory where you will keep your `docker-compose.yaml`.
2. Add the following lines to the `.env` file, replacing the placeholders with your values:
    ```plaintext
    CLAIM_TOKEN=your_claim_token_here
    TZ=your_timezone_here
    MEDIA_LOCATION=your_media_directory_here
    IP_ADDRESS=your_ip_address_here:32400/
    HOSTNAME=your_hostname
    ```

---

## Step 4: Prepare Docker Compose File
1. Create a file named `docker-compose.yaml` in the same directory.
2. Add the following content to the file:
    ```yaml
    version: '3.8'
    services:
      plex:
        container_name: plex
        image: plexinc/pms-docker
        restart: unless-stopped
        ports:
          - 32400:32400/tcp
          - 3005:3005/tcp
          - 8324:8324/tcp
          - 32469:32469/tcp
          - 1900:1900/udp
          - 32410:32410/udp
          - 32412:32412/udp
          - 32413:32413/udp
          - 32414:32414/udp
        environment:
          - PLEX_CLAIM=${CLAIM_TOKEN}
          - TZ=${TZ}
          - PUID=1000  # Replace with your user's UID
          - PGID=1000  # Replace with your user's GID
          - ADVERTISE_IP=http://${IP_ADDRESS}
        hostname: ${HOSTNAME}
        volumes:
          - ./config:/config
          - ./transcode:/transcode
          - ${MEDIA_LOCATION}:/data
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost:32400/web"]
          interval: 1m
          timeout: 10s
          retries: 3
    ```

---

## Step 5: Create Directories
Ensure you have the following directories:
- `config`: Directory for Plex configuration.
- `transcode`: Directory for Plex transcode files.
- Your media directory (as specified in `MEDIA_LOCATION`).

---

## Step 6: Open Port 32400 on Your Router
***THIS PART IS IMPORTANT TO MAKE SURE THE SERVER CAN BE DISCOVERED OUTSIDE OF YOUR LOCAL NETWORK***
1. Log in to your router's admin interface (typically accessed via a web browser at an address like `http://192.168.1.1` or `http://192.168.0.1`).
2. Find the section for port forwarding (this may be under a section named "Advanced", "Firewall", "NAT", or similar).
3. Create a new port forwarding rule with the following settings:
   - External Port: 32400
   - Internal Port: 32400
   - Protocol: TCP
   - Internal IP Address: Your Plex server's IP address (the one obtained in Step 2)

---

## Step 7: Run Docker Compose
1. Open a terminal and navigate to the directory containing `docker-compose.yaml` and `.env`.
2. Run the following command to start the Plex server:
    ```bash
    ./start.sh
    ```
3. (If the permissions didn't work then you may have to run `sudo chmod +x ./start.sh` to grant root access)

---

## Step 8: Access Plex
1. Open a web browser and go to `http://your_ip_address:32400/web`.
2. Log in with your Plex account and start setting up your Plex server.

### Things to make sure when you start setting up plex in the web browser:
- You have to create separate folders for your "Movies" and "TV Shows". Click on the wrench icon and go to the "Manage" section. Click on "Libraries". Then "Add Library". Add the location for your movies and TV shows.
- You also have to properly format your movies so the metadata can properly configure your movies/shows. Format it like this:
    ```
    path/to/media_library/Movies
    ```
    Then in this folder you have a folder for the name of the movie: `Training Day (2001)`.
    Make the format "Title (Year)"!!!!
    Then in that folder you put your .avi, .mp4, .mkv or whatever format in there i.e.:
    `Training_Day_2001_1080p_Torrented1337x.mp4`
- Properly format your shows like this:
    ```
    path/to/media_library/TV Shows
    ```
    Then in this folder you have a folder for the name of the TV show: `Game of Thrones`.
    Then in that folder separate the content into seasons and episodes:
    ```
    Season 1
    Season 2
    Season 3
    ```
    Then in the season folders place your episodes in this format:
    ```
    S01E01.mp4 (Season 1 Episode 1)
    S01E02.mp4 (Season 1 Episode 2)
    ...
    S01E08.mp4 (Season 1 Episode 8)
    ```
    Then format your other folders with the proper episodes.

    Put your .avi, .mp4, .mkv or whatever format in there i.e.:
    `Training_Day_2001_1080p_Torrented1337x.mp4`

- When you want to update your Plex Library with content just click "Scan Library Files".

- You can also click on the Plex icon in the top left corner. Search for the library section you want to update and then just click on the three dots on the right side of that selection. You should see the "Scan Library Files" option there too.

- Run the `.stop.sh` to tear down your Plex server if you want (no real need to do this unless you want to make changes to your container configurations).