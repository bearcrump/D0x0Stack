# Minimal Debian base image
FROM debian:trixie-slim

# Install DOSBox-X and 32-bit libraries
RUN dpkg --add-architecture i386 && \
    apt-get update && \
    apt-get install -y dosbox-x libc6:i386 libstdc++6:i386 && \
    rm -rf /var/lib/apt/lists/*

# Create work directory
WORKDIR /doslab

# Mount the DOS Open Watcom installer from host
# COPY ../open-watcom/open-watcom-c-dos-1.9.exe /doslab/open-watcom-c-dos-1.9.exe

# Make sure the installer is accessible
#RUN chmod +x /doslab/open-watcom-c-dos-1.9.exe

# Default command: launch DOSBox-X in foreground
CMD ["dosbox-x", "-conf", "/doslab/dosbox.conf", "-noconsole"]
