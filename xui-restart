#!/bin/bash

#=================================================================================
#   X-UI Panel Auto-Restart Management Script
#   Created by Web Wizards
#   Version 1.0
#=================================================================================

# Colors for better output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Core variables
CRON_SCRIPT_PATH="/usr/local/bin/xui-cron-restart.sh"
LOG_FILE="/var/log/xui-restart.log"
CRON_COMMENT="XUI_RESTARTER_JOB"

# Check if script is run as root
if [[ $EUID -ne 0 ]]; then
   echo -e "${RED}This script must be run as root.${NC}"
   exit 1
fi

# Function to find the x-ui executable path
find_xui_path() {
    if command -v x-ui &> /dev/null; then
        echo "$(command -v x-ui)"
    elif [ -f "/usr/local/x-ui/x-ui" ]; then
        echo "/usr/local/x-ui/x-ui"
    else
        echo ""
    fi
}

XUI_PATH=$(find_xui_path)

# Installation function
install_restarter() {
    if [ -z "$XUI_PATH" ]; then
        echo -e "${RED}X-UI panel not found! Please ensure the panel is installed correctly.${NC}"
        return
    fi
    
    echo -e "${YELLOW}Enter the restart interval in minutes (e.g., 30 for every 30 minutes):${NC}"
    read -p "Interval in minutes: " interval

    # Validate user input
    if ! [[ "$interval" =~ ^[1-9][0-9]*$ ]]; then
        echo -e "${RED}Invalid input! Please enter a positive integer.${NC}"
        return
    fi

    echo -e "${YELLOW}Creating restart script...${NC}"

    # Create the script that cron will execute
    sudo bash -c "cat > $CRON_SCRIPT_PATH" << EOF
#!/bin/bash
# This script is executed by cron to restart the x-ui panel.

# Restart the x-ui panel
$XUI_PATH restart

# Log the restart event
echo "X-UI panel restarted successfully at \$(date)" >> $LOG_FILE
EOF

    sudo chmod +x "$CRON_SCRIPT_PATH"

    # Remove any existing cron job to prevent duplicates
    (crontab -l 2>/dev/null | grep -v "$CRON_COMMENT") | crontab -

    # Add the new cron job
    (crontab -l 2>/dev/null; echo "*/$interval * * * * $CRON_SCRIPT_PATH # $CRON_COMMENT") | crontab -

    echo -e "${GREEN}===================================================================${NC}"
    echo -e "${GREEN}Installation successful!${NC}"
    echo -e "Your x-ui panel will now automatically restart every ${YELLOW}$interval minutes${NC}."
    echo -e "You can check the logs using option 3 in the menu."
    echo -e "${GREEN}===================================================================${NC}"
}

# Uninstallation function
uninstall_restarter() {
    echo -e "${YELLOW}Are you sure you want to remove the auto-restart schedule? (y/n)${NC}"
    read -p "> " confirm
    if [[ "$confirm" != "y" ]]; then
        echo -e "${RED}Operation cancelled.${NC}"
        return
    fi

    # Remove the cron job
    (crontab -l 2>/dev/null | grep -v "$CRON_COMMENT") | crontab -
    
    # Remove the script file
    if [ -f "$CRON_SCRIPT_PATH" ]; then
        sudo rm "$CRON_SCRIPT_PATH"
    fi

    # Ask to remove the log file
    if [ -f "$LOG_FILE" ]; then
        read -p "Do you also want to delete the log file? (y/n) " delete_log
        if [[ "$delete_log" == "y" ]]; then
            sudo rm "$LOG_FILE"
            echo -e "${GREEN}Log file removed.${NC}"
        fi
    fi

    echo -e "${GREEN}=====================================================${NC}"
    echo -e "${GREEN}Auto-restart schedule has been successfully removed.${NC}"
    echo -e "${GREEN}=====================================================${NC}"
}

# Status check function
check_status() {
    echo -e "${YELLOW}==================== Service Status ====================${NC}"
    cron_job=$(crontab -l 2>/dev/null | grep "$CRON_COMMENT")

    if [ -n "$cron_job" ]; then
        interval=$(echo "$cron_job" | grep -o '*/[0-9]*' | cut -d'/' -f2)
        echo -e "Status: ${GREEN}Active${NC}"
        echo -e "Schedule: Restarts every ${YELLOW}$interval minutes${NC}."
        
        if [ -f "$LOG_FILE" ]; then
            echo -e "\n${YELLOW}--- Last Recorded Restart ---${NC}"
            tail -n 1 "$LOG_FILE"
        else
            echo -e "\n${YELLOW}No logs have been recorded yet.${NC}"
        fi
    else
        echo -e "Status: ${RED}Inactive${NC}"
        echo -e "No auto-restart schedule is configured."
    fi
    echo -e "${YELLOW}====================================================${NC}"
}

# Log viewing function
view_logs() {
    if [ -f "$LOG_FILE" ]; then
        echo -e "${YELLOW}--- Displaying full log file (Press Ctrl+C to exit) ---${NC}"
        cat "$LOG_FILE"
        echo -e "${YELLOW}--- End of logs ---${NC}"
    else
        echo -e "${RED}Log file not found. The service might not be installed or hasn't run yet.${NC}"
    fi
}

# Main menu display function
show_menu() {
    clear
    echo -e "${GREEN}=====================================================${NC}"
    echo -e "      ${YELLOW}X-UI Panel Auto-Restart Manager${NC}"
    echo -e "${GREEN}=====================================================${NC}"
    echo "1. Install or Update Auto-Restart Schedule"
    echo "2. Uninstall Auto-Restart Schedule"
    echo "3. Check Status"
    echo "4. View Restart Logs"
    echo "0. Exit"
    echo -e "${GREEN}=====================================================${NC}"
    read -p "Please select an option: " choice
    
    case $choice in
        1)
            install_restarter
            ;;
        2)
            uninstall_restarter
            ;;
        3)
            check_status
            ;;
        4)
            view_logs
            ;;
        0)
            echo -e "${GREEN}Exiting. Goodbye!${NC}"
            exit 0
            ;;
        *)
            echo -e "${RED}Invalid option! Please try again.${NC}"
            ;;
    esac

    read -p $'\n'"Press Enter to return to the main menu..."
    show_menu
}

# Run the main menu
show_menu
