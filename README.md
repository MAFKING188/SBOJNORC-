# Automated System Updates & Upgrades on Linux

This guide provides multiple methods to automate system updates and upgrades on a Linux-based system. The methods discussed include using `cron`, `anacron`, and `systemd` timers to ensure that your system is always up-to-date, even if the computer is off when the update is scheduled.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Method 1: Using Cron](#method-1-using-cron)
  - [Setup Cron Job](#setup-cron-job)
  - [Handling Missed Cron Jobs](#handling-missed-cron-jobs)
- [Method 2: Using Anacron](#method-2-using-anacron)
  - [Setup Anacron Job](#setup-anacron-job)
- [Method 3: Using Systemd Timers](#method-3-using-systemd-timers)
  - [Create Systemd Service](#create-systemd-service)
  - [Create Systemd Timer](#create-systemd-timer)
  - [Enable and Start the Timer](#enable-and-start-the-timer)
- [How to Store and Use These Methods](#how-to-store-and-use-these-methods)
- [Conclusion](#conclusion)

---

## Prerequisites

- A Linux-based system (e.g., Ubuntu, Debian, etc.).
- Root (sudo) privileges.
- Basic knowledge of using the terminal.
  
---

## Method 1: Using Cron

### Setup Cron Job

To schedule updates using `cron`, follow these steps:

1. Open your crontab file for editing:
   
   ```bash
   crontab -e

----------------------
## Add the following line to schedule daily updates at midnight:

@daily sudo apt update && sudo apt full-upgrade -y

Explanation:

    @daily: Runs the command once a day (typically at midnight).

    sudo apt update: Updates the package lists.

    sudo apt full-upgrade -y: Upgrades all packages and installs/removes as necessary, with -y automatically accepting prompts.

Save and exit the crontab file.

Handling Missed Cron Jobs

If your system is off when the cron job is scheduled to run, the update will be missed. To handle this:

    Option 1: Use @reboot to trigger the update on system startup:

@reboot sudo apt update && sudo apt full-upgrade -y

Option 2: Use anacron for more flexibility (see next method).
--------------------------

## Method 2: Using Anacron

anacron is useful when your system is off at the scheduled time. It will run the missed job after a specified delay when the system starts.
Setup Anacron Job

    Edit the anacrontab file:

sudo nano /etc/anacrontab

Add a new line for the update and upgrade job:

1       0       cron.daily    sudo apt update && sudo apt full-upgrade -y

Explanation:

    1: The job will run once a day.

    0: Delay in days after the job is missed (0 means run immediately on startup).

    cron.daily: The directory where daily tasks are executed.

Save and exit the anacrontab file.
-------------------------

Method 3: Using Systemd Timers

Systemd timers offer flexibility and reliability, especially for modern Linux distributions that use systemd. This method ensures that the update job runs even if the system is off when the scheduled time arrives.
Create Systemd Service

    Create the systemd service file:

sudo nano /etc/systemd/system/apt-update-upgrade.service

Add the following configuration:

    [Unit]
    Description=Run apt update and full upgrade

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/sudo /usr/bin/apt update
    ExecStartPost=/usr/bin/sudo /usr/bin/apt full-upgrade -y

    Explanation:

        ExecStart: Runs the update command.

        ExecStartPost: Runs the upgrade command after the update finishes.

    Save and exit the file.

Create Systemd Timer

    Create the timer file:

sudo nano /etc/systemd/system/apt-update-upgrade.timer

Add the following configuration:

    [Unit]
    Description=Timer for apt update and upgrade

    [Timer]
    OnBootSec=10min
    OnUnitActiveSec=1d
    Unit=apt-update-upgrade.service

    [Install]
    WantedBy=timers.target

    Explanation:

        OnBootSec=10min: Runs the job 10 minutes after the system boots.

        OnUnitActiveSec=1d: Runs the job once per day.

    Save and exit the file.

Enable and Start the Timer

    Reload systemd to apply the changes:

sudo systemctl daemon-reload

Enable the timer to start at boot:

sudo systemctl enable apt-update-upgrade.timer

Start the timer:

sudo systemctl start apt-update-upgrade.timer

Check the timer status:

sudo systemctl status apt-update-upgrade.timer

