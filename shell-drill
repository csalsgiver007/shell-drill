#!/usr/bin/env python3
"""
shell-drill: RHEL 8/9 Terminal Muscle-Memory Trainer
A pure Python 3 curses TUI application designed specifically for Red Hat Enterprise Linux 8 and 9.
Zero external dependencies (uses Python standard library: curses, subprocess, tempfile, json).
Designed to run directly in an SSH or console session on a disposable RHEL 9 snapshot.

Usage:
    python3 shell_drill.py
    ./shell-drill
    ./shell-drill --category "Service Management"
    ./shell-drill --list
    ./shell-drill --library scenarios.json
    ./shell-drill --export-library all_scenarios.json
"""

import os
import sys
import curses
import tempfile
import shutil
import subprocess
import re
import time
import textwrap
import argparse
import json

# ----------------------------------------------------------------------
# Comprehensive Scenario Library (60+ Scenarios for RHEL 8/9)
# ----------------------------------------------------------------------
# Embedded comprehensive scenario bank (230 drills)
# Embedded comprehensive scenario bank (460 drills)
BUILTIN_SCENARIOS = json.loads(r'''[
  {
    "id": "sys-01",
    "category": "Service Management",
    "title": "Enable and Start Service Immediately",
    "difficulty": "Easy",
    "description": "On RHEL 8/9, configure 'sshd' to start at boot and immediately start it now in one command.",
    "objective": "systemctl enable --now sshd",
    "hints": [
      "Use systemctl enable with --now",
      "Command: systemctl enable --now sshd"
    ],
    "solution": "systemctl enable --now sshd",
    "accepted_regex": [
      "systemctl\\s+enable\\s+--now\\s+sshd(\\.service)?",
      "systemctl\\s+--now\\s+enable\\s+sshd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": "systemctl is-enabled sshd 2>/dev/null || systemctl is-active sshd"
  },
  {
    "id": "sys-02",
    "category": "Service Management",
    "title": "Reload systemd Daemon After Unit Edits",
    "difficulty": "Easy",
    "description": "Notify systemd to scan the disk for new or changed unit files without rebooting.",
    "objective": "systemctl daemon-reload",
    "hints": [
      "Use systemctl daemon-reload",
      "Command: systemctl daemon-reload"
    ],
    "solution": "systemctl daemon-reload",
    "accepted_regex": [
      "systemctl\\s+daemon-reload"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-03",
    "category": "Service Management",
    "title": "Query Journald for Priority Errors in Past Hour",
    "difficulty": "Intermediate",
    "description": "Query systemd-journald for 'sshd' logs filtering for priority 'err' from the last 1 hour.",
    "objective": "journalctl -u sshd -p err --since \"1 hour ago\"",
    "hints": [
      "Flags: -u sshd, -p err, --since '1 hour ago'",
      "Command: journalctl -u sshd -p err --since \"1 hour ago\""
    ],
    "solution": "journalctl -u sshd -p err --since \"1 hour ago\"",
    "accepted_regex": [
      "journalctl\\s+.*-u\\s+sshd.*-p\\s+err.*--since\\s+[\"\\']?(-1h|1\\s+hour\\s+ago)[\"\\']?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-04",
    "category": "Service Management",
    "title": "Mask Service to Prevent Accidental Activation",
    "difficulty": "Intermediate",
    "description": "Completely prevent the 'cups' service from being started manually or as a dependency.",
    "objective": "systemctl mask cups",
    "hints": [
      "Use systemctl mask",
      "Command: systemctl mask cups"
    ],
    "solution": "systemctl mask cups",
    "accepted_regex": [
      "systemctl\\s+mask\\s+cups(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-05",
    "category": "Service Management",
    "title": "List All Failed System Units",
    "difficulty": "Easy",
    "description": "List all systemd units currently in a failed or degraded state.",
    "objective": "systemctl --failed",
    "hints": [
      "Command: systemctl --failed"
    ],
    "solution": "systemctl --failed",
    "accepted_regex": [
      "systemctl\\s+--failed",
      "systemctl\\s+list-units\\s+--state=failed"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-06",
    "category": "Service Management",
    "title": "Check If Service Is Actively Running",
    "difficulty": "Easy",
    "description": "Check whether 'sshd' is currently running without verbose output.",
    "objective": "systemctl is-active sshd",
    "hints": [
      "Use is-active verb",
      "Command: systemctl is-active sshd"
    ],
    "solution": "systemctl is-active sshd",
    "accepted_regex": [
      "systemctl\\s+is-active\\s+sshd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-07",
    "category": "Service Management",
    "title": "View Kernel Ring Buffer Logs via Journalctl",
    "difficulty": "Intermediate",
    "description": "View only kernel messages from the current boot using journalctl.",
    "objective": "journalctl -k",
    "hints": [
      "Use -k or --dmesg",
      "Command: journalctl -k"
    ],
    "solution": "journalctl -k",
    "accepted_regex": [
      "journalctl\\s+(-k|--dmesg)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-08",
    "category": "Service Management",
    "title": "Tail Live Service Logs in Follow Mode",
    "difficulty": "Intermediate",
    "description": "Stream live logs for 'sshd', showing last 20 lines and following new output.",
    "objective": "journalctl -u sshd -f -n 20",
    "hints": [
      "Flags: -u sshd, -f, -n 20",
      "Command: journalctl -u sshd -f -n 20"
    ],
    "solution": "journalctl -u sshd -f -n 20",
    "accepted_regex": [
      "journalctl\\s+.*-u\\s+sshd.*(-f\\s+-n\\s+20|-n\\s+20\\s+-f|-fn\\s*20)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-09",
    "category": "Service Management",
    "title": "Set Default Boot Target to Multi-User",
    "difficulty": "Intermediate",
    "description": "Configure default system boot target to multi-user (console mode).",
    "objective": "systemctl set-default multi-user.target",
    "hints": [
      "Command: systemctl set-default multi-user.target"
    ],
    "solution": "systemctl set-default multi-user.target",
    "accepted_regex": [
      "systemctl\\s+set-default\\s+multi-user\\.target"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-10",
    "category": "Service Management",
    "title": "List Active Systemd Timers",
    "difficulty": "Easy",
    "description": "List all systemd timer units on the machine.",
    "objective": "systemctl list-timers",
    "hints": [
      "Command: systemctl list-timers"
    ],
    "solution": "systemctl list-timers",
    "accepted_regex": [
      "systemctl\\s+list-timers(\\s+--all)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-11",
    "category": "Service Management",
    "title": "Inspect Unit Configuration with systemctl cat",
    "difficulty": "Easy",
    "description": "Display configuration file contents and drop-ins for 'sshd.service'.",
    "objective": "systemctl cat sshd",
    "hints": [
      "Command: systemctl cat sshd"
    ],
    "solution": "systemctl cat sshd",
    "accepted_regex": [
      "systemctl\\s+cat\\s+sshd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-12",
    "category": "Service Management",
    "title": "Vacuum Journal Logs to Free Disk Space",
    "difficulty": "Intermediate",
    "description": "Clean up systemd journal logs to retain at most 100 Megabytes.",
    "objective": "journalctl --vacuum-size=100M",
    "hints": [
      "Command: journalctl --vacuum-size=100M"
    ],
    "solution": "journalctl --vacuum-size=100M",
    "accepted_regex": [
      "journalctl\\s+--vacuum-size=100M"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-13",
    "category": "Service Management",
    "title": "Reset Failed State on Cleaned Service",
    "difficulty": "Intermediate",
    "description": "Clear the failed status memory for unit 'httpd'.",
    "objective": "systemctl reset-failed httpd",
    "hints": [
      "Command: systemctl reset-failed httpd"
    ],
    "solution": "systemctl reset-failed httpd",
    "accepted_regex": [
      "systemctl\\s+reset-failed\\s+httpd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-14",
    "category": "Service Management",
    "title": "Query Service SubState Property with systemctl show",
    "difficulty": "Intermediate",
    "description": "Query the 'SubState' property value for unit 'sshd'.",
    "objective": "systemctl show -p SubState sshd",
    "hints": [
      "Command: systemctl show -p SubState sshd"
    ],
    "solution": "systemctl show -p SubState sshd",
    "accepted_regex": [
      "systemctl\\s+show\\s+(-p\\s+SubState|--property=SubState)\\s+sshd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-15",
    "category": "Service Management",
    "title": "Reload or Restart Service Conditionally",
    "difficulty": "Easy",
    "description": "Reload configuration if supported; otherwise restart 'sshd'.",
    "objective": "systemctl reload-or-restart sshd",
    "hints": [
      "Command: systemctl reload-or-restart sshd"
    ],
    "solution": "systemctl reload-or-restart sshd",
    "accepted_regex": [
      "systemctl\\s+reload-or-restart\\s+sshd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-16",
    "category": "Service Management",
    "title": "List Unit Dependency Tree",
    "difficulty": "Easy",
    "description": "Display the visual dependency tree for unit 'sshd.service'.",
    "objective": "systemctl list-dependencies sshd",
    "hints": [
      "Command: systemctl list-dependencies sshd"
    ],
    "solution": "systemctl list-dependencies sshd",
    "accepted_regex": [
      "systemctl\\s+list-dependencies\\s+sshd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-17",
    "category": "Service Management",
    "title": "Filter Journal Logs by Root User UID",
    "difficulty": "Intermediate",
    "description": "Query journald for messages initiated by root (UID 0), showing last 25 lines.",
    "objective": "journalctl _UID=0 -n 25",
    "hints": [
      "Command: journalctl _UID=0 -n 25"
    ],
    "solution": "journalctl _UID=0 -n 25",
    "accepted_regex": [
      "journalctl\\s+.*_UID=0.*-n\\s+25",
      "journalctl\\s+.*-n\\s+25.*_UID=0"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-18",
    "category": "Service Management",
    "title": "View Logs from Previous System Boot",
    "difficulty": "Intermediate",
    "description": "Query journalctl for logs from the prior boot session (offset -1).",
    "objective": "journalctl -b -1",
    "hints": [
      "Command: journalctl -b -1"
    ],
    "solution": "journalctl -b -1",
    "accepted_regex": [
      "journalctl\\s+(-b\\s+-1|--boot=-1)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-19",
    "category": "Service Management",
    "title": "Check If Service Is Enabled for Boot",
    "difficulty": "Easy",
    "description": "Check if 'crond' is set to enable at system boot.",
    "objective": "systemctl is-enabled crond",
    "hints": [
      "Command: systemctl is-enabled crond"
    ],
    "solution": "systemctl is-enabled crond",
    "accepted_regex": [
      "systemctl\\s+is-enabled\\s+crond(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-20",
    "category": "Service Management",
    "title": "Unmask a Masked Service",
    "difficulty": "Intermediate",
    "description": "Restore the 'cups' service from masked state so it can be used again.",
    "objective": "systemctl unmask cups",
    "hints": [
      "Command: systemctl unmask cups"
    ],
    "solution": "systemctl unmask cups",
    "accepted_regex": [
      "systemctl\\s+unmask\\s+cups(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-21",
    "category": "Service Management",
    "title": "Disable Service and Stop It Immediately",
    "difficulty": "Easy",
    "description": "Disable 'postfix' from starting at boot and stop it right now in one command.",
    "objective": "systemctl disable --now postfix",
    "hints": [
      "Command: systemctl disable --now postfix"
    ],
    "solution": "systemctl disable --now postfix",
    "accepted_regex": [
      "systemctl\\s+(disable\\s+--now|--now\\s+disable)\\s+postfix(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-22",
    "category": "Service Management",
    "title": "Isolate Graphical Target On Demand",
    "difficulty": "Intermediate",
    "description": "Switch active operating state to graphical.target immediately.",
    "objective": "systemctl isolate graphical.target",
    "hints": [
      "Command: systemctl isolate graphical.target"
    ],
    "solution": "systemctl isolate graphical.target",
    "accepted_regex": [
      "systemctl\\s+isolate\\s+graphical\\.target"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-23",
    "category": "Service Management",
    "title": "List All Installed Service Unit Files",
    "difficulty": "Easy",
    "description": "Display state of all unit files (enabled, disabled, static, masked).",
    "objective": "systemctl list-unit-files --type=service",
    "hints": [
      "Command: systemctl list-unit-files --type=service"
    ],
    "solution": "systemctl list-unit-files --type=service",
    "accepted_regex": [
      "systemctl\\s+list-unit-files\\s+--type=service"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-24",
    "category": "Service Management",
    "title": "Vacuum Journal Logs by Retention Time",
    "difficulty": "Intermediate",
    "description": "Purge journal archives older than 2 days.",
    "objective": "journalctl --vacuum-time=2d",
    "hints": [
      "Command: journalctl --vacuum-time=2d"
    ],
    "solution": "journalctl --vacuum-time=2d",
    "accepted_regex": [
      "journalctl\\s+--vacuum-time=2d"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-25",
    "category": "Service Management",
    "title": "Display Current Disk Usage of Journal Logs",
    "difficulty": "Easy",
    "description": "Check how much storage space is currently consumed by systemd-journald.",
    "objective": "journalctl --disk-usage",
    "hints": [
      "Command: journalctl --disk-usage"
    ],
    "solution": "journalctl --disk-usage",
    "accepted_regex": [
      "journalctl\\s+--disk-usage"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-26",
    "category": "Service Management",
    "title": "Filter Journal by Specific Process ID",
    "difficulty": "Intermediate",
    "description": "Query logs generated specifically by PID 1234.",
    "objective": "journalctl _PID=1234",
    "hints": [
      "Command: journalctl _PID=1234"
    ],
    "solution": "journalctl _PID=1234",
    "accepted_regex": [
      "journalctl\\s+_PID=1234"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-27",
    "category": "Service Management",
    "title": "Output Journal Entries in Pretty JSON",
    "difficulty": "Intermediate",
    "description": "Query last 5 logs for 'sshd' formatted in structured json-pretty.",
    "objective": "journalctl -u sshd -n 5 -o json-pretty",
    "hints": [
      "Command: journalctl -u sshd -n 5 -o json-pretty"
    ],
    "solution": "journalctl -u sshd -n 5 -o json-pretty",
    "accepted_regex": [
      "journalctl\\s+.*-u\\s+sshd.*(-o\\s+json-pretty|--output=json-pretty)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-28",
    "category": "Service Management",
    "title": "Check Current Default Boot Target",
    "difficulty": "Easy",
    "description": "Print the currently configured default target without modifying it.",
    "objective": "systemctl get-default",
    "hints": [
      "Command: systemctl get-default"
    ],
    "solution": "systemctl get-default",
    "accepted_regex": [
      "systemctl\\s+get-default"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-29",
    "category": "Service Management",
    "title": "Kill Specific Service with SIGTERM",
    "difficulty": "Intermediate",
    "description": "Send SIGTERM kill signal to all processes under unit 'httpd'.",
    "objective": "systemctl kill httpd",
    "hints": [
      "Command: systemctl kill httpd"
    ],
    "solution": "systemctl kill httpd",
    "accepted_regex": [
      "systemctl\\s+kill\\s+httpd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-30",
    "category": "Service Management",
    "title": "List All Active Socket Units",
    "difficulty": "Easy",
    "description": "List all systemd socket units and their listening endpoints.",
    "objective": "systemctl list-sockets",
    "hints": [
      "Command: systemctl list-sockets"
    ],
    "solution": "systemctl list-sockets",
    "accepted_regex": [
      "systemctl\\s+list-sockets"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-31",
    "category": "Service Management",
    "title": "Query Journal Logs for System Boot Events",
    "difficulty": "Easy",
    "description": "Show all log events recorded during the current boot from start to finish.",
    "objective": "journalctl -b",
    "hints": [
      "Command: journalctl -b"
    ],
    "solution": "journalctl -b",
    "accepted_regex": [
      "journalctl\\s+(-b|--boot)(\\s+0)?$"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-32",
    "category": "Service Management",
    "title": "Show Only Service Description and Path",
    "difficulty": "Intermediate",
    "description": "Display the Description and FragmentPath properties of unit 'sshd'.",
    "objective": "systemctl show -p Description,FragmentPath sshd",
    "hints": [
      "Command: systemctl show -p Description,FragmentPath sshd"
    ],
    "solution": "systemctl show -p Description,FragmentPath sshd",
    "accepted_regex": [
      "systemctl\\s+show\\s+(-p|--property=).*sshd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-33",
    "category": "Service Management",
    "title": "Restart Failed NetworkManager Service",
    "difficulty": "Easy",
    "description": "Restart the NetworkManager service daemon.",
    "objective": "systemctl restart NetworkManager",
    "hints": [
      "Command: systemctl restart NetworkManager"
    ],
    "solution": "systemctl restart NetworkManager",
    "accepted_regex": [
      "systemctl\\s+restart\\s+NetworkManager(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-34",
    "category": "Service Management",
    "title": "Inspect Journal Logs Between Exact Time Window",
    "difficulty": "Intermediate",
    "description": "Query logs recorded between '2026-10-01 00:00:00' and '2026-10-01 12:00:00'.",
    "objective": "journalctl --since \"2026-10-01 00:00:00\" --until \"2026-10-01 12:00:00\"",
    "hints": [
      "Command: journalctl --since \"2026-10-01 00:00:00\" --until \"2026-10-01 12:00:00\""
    ],
    "solution": "journalctl --since \"2026-10-01 00:00:00\" --until \"2026-10-01 12:00:00\"",
    "accepted_regex": [
      "journalctl\\s+.*--since\\s+[\"\\']2026-10-01.*--until\\s+[\"\\']2026-10-01"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-35",
    "category": "Service Management",
    "title": "Audit Failed Status of Specific Service",
    "difficulty": "Easy",
    "description": "Test if 'mariadb' is in a failed state using is-failed.",
    "objective": "systemctl is-failed mariadb",
    "hints": [
      "Command: systemctl is-failed mariadb"
    ],
    "solution": "systemctl is-failed mariadb",
    "accepted_regex": [
      "systemctl\\s+is-failed\\s+mariadb(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-36",
    "category": "Service Management",
    "title": "View Explanation Help with journalctl -xe",
    "difficulty": "Intermediate",
    "description": "View recent errors with catalog explanations jump to end.",
    "objective": "journalctl -xe",
    "hints": [
      "Command: journalctl -xe"
    ],
    "solution": "journalctl -xe",
    "accepted_regex": [
      "journalctl\\s+(-xe|-ex)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-37",
    "category": "Service Management",
    "title": "Show Inactive Services on Host",
    "difficulty": "Intermediate",
    "description": "List service units that are currently in inactive state.",
    "objective": "systemctl list-units --type=service --state=inactive",
    "hints": [
      "Command: systemctl list-units --type=service --state=inactive"
    ],
    "solution": "systemctl list-units --type=service --state=inactive",
    "accepted_regex": [
      "systemctl\\s+list-units\\s+--type=service\\s+--state=inactive"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-38",
    "category": "Service Management",
    "title": "Verify Journalctl Log Priority Warning and Above",
    "difficulty": "Intermediate",
    "description": "Query logs with priority warning (level 4) or higher.",
    "objective": "journalctl -p warning",
    "hints": [
      "Command: journalctl -p warning"
    ],
    "solution": "journalctl -p warning",
    "accepted_regex": [
      "journalctl\\s+.*-p\\s+(warning|4)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-39",
    "category": "Service Management",
    "title": "Check systemd System State Overall",
    "difficulty": "Easy",
    "description": "Check if overall system manager status is running, degraded, or maintenance.",
    "objective": "systemctl is-system-running",
    "hints": [
      "Command: systemctl is-system-running"
    ],
    "solution": "systemctl is-system-running",
    "accepted_regex": [
      "systemctl\\s+is-system-running"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-40",
    "category": "Service Management",
    "title": "Query Unit Journal Logs Without Pagination",
    "difficulty": "Easy",
    "description": "Print logs for 'firewalld' to terminal without piping into less/pager.",
    "objective": "journalctl -u firewalld --no-pager",
    "hints": [
      "Command: journalctl -u firewalld --no-pager"
    ],
    "solution": "journalctl -u firewalld --no-pager",
    "accepted_regex": [
      "journalctl\\s+.*-u\\s+firewalld.*--no-pager",
      "journalctl\\s+.*--no-pager.*-u\\s+firewalld"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-01",
    "category": "Firewall & Network",
    "title": "Permanently Allow HTTPS Service in Public Zone",
    "difficulty": "Easy",
    "description": "Allow inbound HTTPS traffic in default public zone across reboots.",
    "objective": "firewall-cmd --permanent --zone=public --add-service=https",
    "hints": [
      "Command: firewall-cmd --permanent --zone=public --add-service=https"
    ],
    "solution": "firewall-cmd --permanent --zone=public --add-service=https",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--zone=public.*--add-service=https",
      "firewall-cmd\\s+.*--add-service=https.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-02",
    "category": "Firewall & Network",
    "title": "Reload Firewalld Without Dropping Connections",
    "difficulty": "Easy",
    "description": "Apply permanent firewall rules into live runtime without disconnects.",
    "objective": "firewall-cmd --reload",
    "hints": [
      "Command: firewall-cmd --reload"
    ],
    "solution": "firewall-cmd --reload",
    "accepted_regex": [
      "firewall-cmd\\s+--reload"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-03",
    "category": "Firewall & Network",
    "title": "Audit Active Firewall Rules",
    "difficulty": "Easy",
    "description": "Inspect active firewall state for public default zone.",
    "objective": "firewall-cmd --list-all",
    "hints": [
      "Command: firewall-cmd --list-all"
    ],
    "solution": "firewall-cmd --list-all",
    "accepted_regex": [
      "firewall-cmd\\s+--list-all(\\s+--zone=public)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-04",
    "category": "Firewall & Network",
    "title": "Open Custom TCP Port 8080 Permanently",
    "difficulty": "Intermediate",
    "description": "Allow inbound traffic on custom port 8080/tcp permanently.",
    "objective": "firewall-cmd --permanent --add-port=8080/tcp",
    "hints": [
      "Command: firewall-cmd --permanent --add-port=8080/tcp"
    ],
    "solution": "firewall-cmd --permanent --add-port=8080/tcp",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-port=8080\\/tcp",
      "firewall-cmd\\s+.*--add-port=8080\\/tcp.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-05",
    "category": "Firewall & Network",
    "title": "Remove Insecure Service Permanently from Firewall",
    "difficulty": "Intermediate",
    "description": "Remove 'cockpit' service from permanent firewall configuration.",
    "objective": "firewall-cmd --permanent --zone=public --remove-service=cockpit",
    "hints": [
      "Command: firewall-cmd --permanent --zone=public --remove-service=cockpit"
    ],
    "solution": "firewall-cmd --permanent --zone=public --remove-service=cockpit",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--remove-service=cockpit",
      "firewall-cmd\\s+.*--remove-service=cockpit.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-06",
    "category": "Firewall & Network",
    "title": "Enable IP Masquerading Permanently",
    "difficulty": "Intermediate",
    "description": "Enable IP masquerading (NAT) permanently in public zone.",
    "objective": "firewall-cmd --permanent --zone=public --add-masquerade",
    "hints": [
      "Command: firewall-cmd --permanent --zone=public --add-masquerade"
    ],
    "solution": "firewall-cmd --permanent --zone=public --add-masquerade",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-masquerade",
      "firewall-cmd\\s+.*--add-masquerade.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-07",
    "category": "Firewall & Network",
    "title": "Add Port Forwarding Rule in Firewalld",
    "difficulty": "Advanced",
    "description": "Permanently forward incoming port 80 traffic to internal port 8080.",
    "objective": "firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080",
    "hints": [
      "Command: firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080"
    ],
    "solution": "firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-forward-port=port=80:proto=tcp:toport=8080"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-08",
    "category": "Firewall & Network",
    "title": "Check Running Status of Firewalld",
    "difficulty": "Easy",
    "description": "Check whether firewalld is running using firewall-cmd --state.",
    "objective": "firewall-cmd --state",
    "hints": [
      "Command: firewall-cmd --state"
    ],
    "solution": "firewall-cmd --state",
    "accepted_regex": [
      "firewall-cmd\\s+--state"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-09",
    "category": "Firewall & Network",
    "title": "Get Default Firewall Zone",
    "difficulty": "Easy",
    "description": "Print the name of default firewalld zone.",
    "objective": "firewall-cmd --get-default-zone",
    "hints": [
      "Command: firewall-cmd --get-default-zone"
    ],
    "solution": "firewall-cmd --get-default-zone",
    "accepted_regex": [
      "firewall-cmd\\s+--get-default-zone"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-10",
    "category": "Firewall & Network",
    "title": "Set Default Firewall Zone to Internal",
    "difficulty": "Intermediate",
    "description": "Change the default zone to 'internal'.",
    "objective": "firewall-cmd --set-default-zone=internal",
    "hints": [
      "Command: firewall-cmd --set-default-zone=internal"
    ],
    "solution": "firewall-cmd --set-default-zone=internal",
    "accepted_regex": [
      "firewall-cmd\\s+--set-default-zone=internal"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-11",
    "category": "Firewall & Network",
    "title": "List All Available Firewall Services",
    "difficulty": "Easy",
    "description": "List all predefined services supported by firewalld.",
    "objective": "firewall-cmd --get-services",
    "hints": [
      "Command: firewall-cmd --get-services"
    ],
    "solution": "firewall-cmd --get-services",
    "accepted_regex": [
      "firewall-cmd\\s+--get-services"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-12",
    "category": "Firewall & Network",
    "title": "Permanently Restrict SSH to Source Subnet",
    "difficulty": "Advanced",
    "description": "Add permanent rich rule permitting SSH only from subnet 10.0.0.0/24.",
    "objective": "firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"10.0.0.0/24\" service name=\"ssh\" accept'",
    "hints": [
      "Command: firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"10.0.0.0/24\" service name=\"ssh\" accept'"
    ],
    "solution": "firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"10.0.0.0/24\" service name=\"ssh\" accept'",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-rich-rule=.*10\\.0\\.0\\.0\\/24.*ssh.*accept"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-13",
    "category": "Firewall & Network",
    "title": "Remove Open Port from Permanent Firewall",
    "difficulty": "Intermediate",
    "description": "Remove port 8080/tcp from permanent public configuration.",
    "objective": "firewall-cmd --permanent --remove-port=8080/tcp",
    "hints": [
      "Command: firewall-cmd --permanent --remove-port=8080/tcp"
    ],
    "solution": "firewall-cmd --permanent --remove-port=8080/tcp",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--remove-port=8080\\/tcp",
      "firewall-cmd\\s+.*--remove-port=8080\\/tcp.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-14",
    "category": "Firewall & Network",
    "title": "Query If Service Is Permitted in Firewall",
    "difficulty": "Easy",
    "description": "Check if 'http' service is currently enabled in public zone (returns 0 or 1).",
    "objective": "firewall-cmd --zone=public --query-service=http",
    "hints": [
      "Command: firewall-cmd --zone=public --query-service=http"
    ],
    "solution": "firewall-cmd --zone=public --query-service=http",
    "accepted_regex": [
      "firewall-cmd\\s+.*--query-service=http"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-15",
    "category": "Firewall & Network",
    "title": "Bind Network Interface to Trusted Zone Permanently",
    "difficulty": "Intermediate",
    "description": "Assign interface 'ens224' to 'trusted' zone permanently.",
    "objective": "firewall-cmd --permanent --zone=trusted --add-interface=ens224",
    "hints": [
      "Command: firewall-cmd --permanent --zone=trusted --add-interface=ens224"
    ],
    "solution": "firewall-cmd --permanent --zone=trusted --add-interface=ens224",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--zone=trusted.*--add-interface=ens224",
      "firewall-cmd\\s+.*--zone=trusted.*--add-interface=ens224.*--permanent"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-16",
    "category": "Firewall & Network",
    "title": "List Active Firewall Zones and Interfaces",
    "difficulty": "Easy",
    "description": "Print only active zones and their bound network interfaces.",
    "objective": "firewall-cmd --get-active-zones",
    "hints": [
      "Command: firewall-cmd --get-active-zones"
    ],
    "solution": "firewall-cmd --get-active-zones",
    "accepted_regex": [
      "firewall-cmd\\s+--get-active-zones"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-17",
    "category": "Firewall & Network",
    "title": "Audit Rules Across All Zones",
    "difficulty": "Intermediate",
    "description": "Display rules and configurations for all existing zones.",
    "objective": "firewall-cmd --list-all-zones",
    "hints": [
      "Command: firewall-cmd --list-all-zones"
    ],
    "solution": "firewall-cmd --list-all-zones",
    "accepted_regex": [
      "firewall-cmd\\s+--list-all-zones"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-18",
    "category": "Firewall & Network",
    "title": "Temporarily Add Service to Running State",
    "difficulty": "Easy",
    "description": "Add 'nfs' to public zone temporarily without --permanent flag.",
    "objective": "firewall-cmd --zone=public --add-service=nfs",
    "hints": [
      "Command: firewall-cmd --zone=public --add-service=nfs"
    ],
    "solution": "firewall-cmd --zone=public --add-service=nfs",
    "accepted_regex": [
      "firewall-cmd\\s+(--zone=public\\s+)?--add-service=nfs$"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-19",
    "category": "Firewall & Network",
    "title": "Query Current SELinux Enforcement Mode",
    "difficulty": "Easy",
    "description": "Print current SELinux status using getenforce.",
    "objective": "getenforce",
    "hints": [
      "Command: getenforce"
    ],
    "solution": "getenforce",
    "accepted_regex": [
      "^getenforce$"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-20",
    "category": "Firewall & Network",
    "title": "Switch SELinux Temporarily to Permissive",
    "difficulty": "Easy",
    "description": "Set SELinux runtime mode to Permissive (0) without rebooting.",
    "objective": "setenforce 0",
    "hints": [
      "Command: setenforce 0"
    ],
    "solution": "setenforce 0",
    "accepted_regex": [
      "^setenforce\\s+(0|Permissive)$"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-21",
    "category": "Firewall & Network",
    "title": "Check Comprehensive SELinux Status",
    "difficulty": "Easy",
    "description": "Display detailed SELinux operational status with sestatus.",
    "objective": "sestatus",
    "hints": [
      "Command: sestatus"
    ],
    "solution": "sestatus",
    "accepted_regex": [
      "^sestatus$"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-22",
    "category": "Firewall & Network",
    "title": "Restore Default SELinux File Context Recursively",
    "difficulty": "Intermediate",
    "description": "Restore default SELinux context on /var/www/html recursively with verbose output.",
    "objective": "restorecon -Rv /var/www/html",
    "hints": [
      "Command: restorecon -Rv /var/www/html"
    ],
    "solution": "restorecon -Rv /var/www/html",
    "accepted_regex": [
      "restorecon\\s+(-Rv|-vR|-R\\s+-v|-v\\s+-R)\\s+\\/var\\/www\\/html"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-23",
    "category": "Firewall & Network",
    "title": "Set Persistent SELinux Boolean Value",
    "difficulty": "Intermediate",
    "description": "Enable boolean 'httpd_can_network_connect' persistently across reboots.",
    "objective": "setsebool -P httpd_can_network_connect on",
    "hints": [
      "Use setsebool -P with on or 1",
      "Command: setsebool -P httpd_can_network_connect on"
    ],
    "solution": "setsebool -P httpd_can_network_connect on",
    "accepted_regex": [
      "setsebool\\s+-P\\s+httpd_can_network_connect\\s+(on|1)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-24",
    "category": "Firewall & Network",
    "title": "Query State of Specific SELinux Boolean",
    "difficulty": "Easy",
    "description": "Check current and pending state of boolean 'httpd_enable_homedirs'.",
    "objective": "getsebool httpd_enable_homedirs",
    "hints": [
      "Command: getsebool httpd_enable_homedirs"
    ],
    "solution": "getsebool httpd_enable_homedirs",
    "accepted_regex": [
      "getsebool\\s+httpd_enable_homedirs"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-25",
    "category": "Firewall & Network",
    "title": "Display Listening TCP Sockets with ss",
    "difficulty": "Intermediate",
    "description": "Display all listening (-l) TCP (-t) sockets with numeric ports (-n) and processes (-p).",
    "objective": "ss -tlpn",
    "hints": [
      "Command: ss -tlpn"
    ],
    "solution": "ss -tlpn",
    "accepted_regex": [
      "ss\\s+-(tlpn|tulpn|tnlp|ltpn|lntp)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-26",
    "category": "Firewall & Network",
    "title": "Bring Up NetworkManager Connection",
    "difficulty": "Easy",
    "description": "Activate NetworkManager connection profile 'ens192'.",
    "objective": "nmcli con up ens192",
    "hints": [
      "Command: nmcli con up ens192"
    ],
    "solution": "nmcli con up ens192",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+up\\s+([a-zA-Z0-9_-]+)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-27",
    "category": "Firewall & Network",
    "title": "List Active NetworkManager Connections",
    "difficulty": "Easy",
    "description": "Display only currently active NetworkManager connections.",
    "objective": "nmcli con show --active",
    "hints": [
      "Command: nmcli con show --active"
    ],
    "solution": "nmcli con show --active",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+show\\s+--active"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-28",
    "category": "Firewall & Network",
    "title": "Display Network Device Status Overview",
    "difficulty": "Easy",
    "description": "Display status of all network hardware devices with nmcli.",
    "objective": "nmcli dev status",
    "hints": [
      "Command: nmcli dev status"
    ],
    "solution": "nmcli dev status",
    "accepted_regex": [
      "nmcli\\s+(device|dev)\\s+status"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-29",
    "category": "Firewall & Network",
    "title": "Set Static DNS Servers on Connection",
    "difficulty": "Intermediate",
    "description": "Configure connection 'ens192' with DNS servers 8.8.8.8 and 1.1.1.1.",
    "objective": "nmcli con mod ens192 ipv4.dns \"8.8.8.8 1.1.1.1\"",
    "hints": [
      "Command: nmcli con mod ens192 ipv4.dns \"8.8.8.8 1.1.1.1\""
    ],
    "solution": "nmcli con mod ens192 ipv4.dns \"8.8.8.8 1.1.1.1\"",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+mod(ify)?\\s+ens192\\s+ipv4\\.dns\\s+[\"\\']?8\\.8\\.8\\.8"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-30",
    "category": "Firewall & Network",
    "title": "Brief View of All Interface IP Addresses",
    "difficulty": "Easy",
    "description": "Display concise one-line overview of all network interfaces and IPs.",
    "objective": "ip -br a",
    "hints": [
      "Command: ip -br a"
    ],
    "solution": "ip -br a",
    "accepted_regex": [
      "ip\\s+(-br|-brief)\\s+(a|addr|address)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-31",
    "category": "Firewall & Network",
    "title": "Inspect Default Kernel Gateway Route",
    "difficulty": "Easy",
    "description": "Display only the default route entry from routing table.",
    "objective": "ip route show default",
    "hints": [
      "Command: ip route show default"
    ],
    "solution": "ip route show default",
    "accepted_regex": [
      "ip\\s+(route|r)\\s+(show\\s+)?default",
      "ip\\s+(route|r)\\s*\\|\\s*grep\\s+default"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-32",
    "category": "Firewall & Network",
    "title": "Display Kernel ARP Neighbor Table",
    "difficulty": "Easy",
    "description": "Inspect Linux kernel neighbor/ARP table using iproute2.",
    "objective": "ip neigh show",
    "hints": [
      "Command: ip neigh show"
    ],
    "solution": "ip neigh show",
    "accepted_regex": [
      "ip\\s+(neigh(bor)?\\s+(show)?|n)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-33",
    "category": "Firewall & Network",
    "title": "Display Socket Summary Statistics with ss",
    "difficulty": "Easy",
    "description": "Print summary statistics for total, TCP, UDP sockets.",
    "objective": "ss -s",
    "hints": [
      "Command: ss -s"
    ],
    "solution": "ss -s",
    "accepted_regex": [
      "ss\\s+(-s|--summary)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-34",
    "category": "Firewall & Network",
    "title": "Ping Remote Host with 3 Count and 2s Timeout",
    "difficulty": "Easy",
    "description": "Send exactly 3 ICMP echo requests with 2-second timeout to 192.168.1.1.",
    "objective": "ping -c 3 -W 2 192.168.1.1",
    "hints": [
      "Command: ping -c 3 -W 2 192.168.1.1"
    ],
    "solution": "ping -c 3 -W 2 192.168.1.1",
    "accepted_regex": [
      "ping\\s+.*-c\\s+3.*(-W\\s+2)?.*192\\.168\\.1\\.1"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-35",
    "category": "Firewall & Network",
    "title": "Check HTTP Response Headers with Curl",
    "difficulty": "Easy",
    "description": "Fetch only HTTP headers (HEAD request) from http://localhost.",
    "objective": "curl -I http://localhost",
    "hints": [
      "Command: curl -I http://localhost"
    ],
    "solution": "curl -I http://localhost",
    "accepted_regex": [
      "curl\\s+(-I|--head)\\s+http:\\/\\/localhost"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-36",
    "category": "Firewall & Network",
    "title": "Configure Static IPv4 Address on Connection",
    "difficulty": "Intermediate",
    "description": "Set static IP 192.168.1.50/24 with manual method on 'ens192'.",
    "objective": "nmcli con mod ens192 ipv4.addresses 192.168.1.50/24 ipv4.method manual",
    "hints": [
      "Command: nmcli con mod ens192 ipv4.addresses 192.168.1.50/24 ipv4.method manual"
    ],
    "solution": "nmcli con mod ens192 ipv4.addresses 192.168.1.50/24 ipv4.method manual",
    "accepted_regex": [
      "nmcli\\s+.*con\\s+mod.*ens192.*ipv4\\.addresses\\s+192\\.168\\.1\\.50\\/24.*ipv4\\.method\\s+manual"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-37",
    "category": "Firewall & Network",
    "title": "Add New Ethernet Connection Profile",
    "difficulty": "Intermediate",
    "description": "Create a new ethernet connection profile named 'con-ens224' for device ens224.",
    "objective": "nmcli con add type ethernet con-name con-ens224 ifname ens224",
    "hints": [
      "Command: nmcli con add type ethernet con-name con-ens224 ifname ens224"
    ],
    "solution": "nmcli con add type ethernet con-name con-ens224 ifname ens224",
    "accepted_regex": [
      "nmcli\\s+con\\s+add\\s+type\\s+ethernet\\s+con-name\\s+con-ens224\\s+ifname\\s+ens224"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-38",
    "category": "Firewall & Network",
    "title": "Reload NetworkManager Connection Files from Disk",
    "difficulty": "Easy",
    "description": "Instruct NetworkManager to reload connection keyfiles from /etc/NetworkManager/system-connections/.",
    "objective": "nmcli con reload",
    "hints": [
      "Command: nmcli con reload"
    ],
    "solution": "nmcli con reload",
    "accepted_regex": [
      "nmcli\\s+con\\s+reload"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-39",
    "category": "Firewall & Network",
    "title": "Check Socket Listening on Specific Port 3306",
    "difficulty": "Intermediate",
    "description": "Filter ss output to check if port 3306 is listening.",
    "objective": "ss -tlpn sport = :3306",
    "hints": [
      "Command: ss -tlpn sport = :3306 or ss -tlpn | grep 3306"
    ],
    "solution": "ss -tlpn sport = :3306",
    "accepted_regex": [
      "ss\\s+.*3306",
      "ss\\s+-tlpn\\s*\\|\\s*grep\\s+3306"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-40",
    "category": "Firewall & Network",
    "title": "Display Detailed Link Statistics with ip -s",
    "difficulty": "Easy",
    "description": "Display packet transfer and error statistics for interface ens192.",
    "objective": "ip -s link show ens192",
    "hints": [
      "Command: ip -s link show ens192"
    ],
    "solution": "ip -s link show ens192",
    "accepted_regex": [
      "ip\\s+-s\\s+link\\s+show\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-01",
    "category": "Grep & Regex",
    "title": "Filter HTTP 500 Server Errors with Line Numbers",
    "difficulty": "Easy",
    "description": "Extract lines containing status code 500 with line numbers from httpd_access.log.",
    "objective": "grep -n \" 500 \" httpd_access.log",
    "hints": [
      "Command: grep -n \" 500 \" httpd_access.log"
    ],
    "solution": "grep -n \" 500 \" httpd_access.log",
    "accepted_regex": [
      "grep\\s+(-n\\s+[\"\\']?\\s*500\\s*[\"\\']?|[\"\\']?\\s*500\\s*[\"\\']?\\s+-n)\\s+httpd_access\\.log"
    ],
    "setup_files": {
      "httpd_access.log": "10.0.0.1 - [date] \"GET / HTTP/1.1\" 200 120\n10.0.0.2 - [date] \"POST /api HTTP/1.1\" 500 512\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-02",
    "category": "Grep & Regex",
    "title": "Invert Match Comment & Empty Lines",
    "difficulty": "Intermediate",
    "description": "Filter out lines starting with '#' and empty lines from sshd_config.test.",
    "objective": "grep -v -E '^(#|$)' sshd_config.test",
    "hints": [
      "Command: grep -v -E '^(#|$)' sshd_config.test"
    ],
    "solution": "grep -v -E '^(#|$)' sshd_config.test",
    "accepted_regex": [
      "grep\\s+(-vE|-Ev|-E\\s+-v|-v\\s+-E)\\s+['\\\"]?\\^(\\#|\\$)[|](\\#|\\$)['\\\"]?\\s+sshd_config\\.test"
    ],
    "setup_files": {
      "sshd_config.test": "# comment\nPort 22\n\nPermitRootLogin no\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-03",
    "category": "Grep & Regex",
    "title": "Count Failed SSH Password Log Occurrences",
    "difficulty": "Easy",
    "description": "Count matching lines recording 'Failed password' in auth.log.",
    "objective": "grep -c \"Failed password\" auth.log",
    "hints": [
      "Command: grep -c \"Failed password\" auth.log"
    ],
    "solution": "grep -c \"Failed password\" auth.log",
    "accepted_regex": [
      "grep\\s+-c\\s+[\"\\']Failed password[\"\\']\\s+auth\\.log"
    ],
    "setup_files": {
      "auth.log": "Failed password for root\nAccepted publickey\nFailed password for admin\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-04",
    "category": "Grep & Regex",
    "title": "Match Exact Whole Word Using Boundary",
    "difficulty": "Intermediate",
    "description": "Search services.txt for exact word 'root' (not chroot or rootfs).",
    "objective": "grep -w 'root' services.txt",
    "hints": [
      "Command: grep -w 'root' services.txt"
    ],
    "solution": "grep -w 'root' services.txt",
    "accepted_regex": [
      "grep\\s+-w\\s+['\\\"]?root['\\\"]?\\s+services\\.txt"
    ],
    "setup_files": {
      "services.txt": "user root\nuser chroot\nuser rootfs\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-05",
    "category": "Grep & Regex",
    "title": "Display Lines with Context Surrounding Match",
    "difficulty": "Intermediate",
    "description": "Search app.log for 'FATAL' showing 2 lines before and after.",
    "objective": "grep -C 2 'FATAL' app.log",
    "hints": [
      "Command: grep -C 2 'FATAL' app.log"
    ],
    "solution": "grep -C 2 'FATAL' app.log",
    "accepted_regex": [
      "grep\\s+(-C\\s*2|-B\\s*2\\s+-A\\s*2)\\s+['\\\"]?FATAL['\\\"]?\\s+app\\.log"
    ],
    "setup_files": {
      "app.log": "line1\nline2\nFATAL: crash\nline4\nline5\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-06",
    "category": "Grep & Regex",
    "title": "List Only Filenames Containing Match",
    "difficulty": "Easy",
    "description": "Search *.conf for 'AllowOverride' printing only matching filenames.",
    "objective": "grep -l 'AllowOverride' *.conf",
    "hints": [
      "Command: grep -l 'AllowOverride' *.conf"
    ],
    "solution": "grep -l 'AllowOverride' *.conf",
    "accepted_regex": [
      "grep\\s+-l\\s+['\\\"]?AllowOverride['\\\"]?\\s+\\*\\.conf"
    ],
    "setup_files": {
      "a.conf": "AllowOverride All\n",
      "b.conf": "None\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-07",
    "category": "Grep & Regex",
    "title": "Match Valid IPv4 Address Lines with ERE",
    "difficulty": "Advanced",
    "description": "Search nodes.txt for lines starting with an IPv4 address.",
    "objective": "grep -E '^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' nodes.txt",
    "hints": [
      "Command: grep -E '^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' nodes.txt"
    ],
    "solution": "grep -E '^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' nodes.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]\\^\\[0-9\\]\\{1,3\\}"
    ],
    "setup_files": {
      "nodes.txt": "192.168.1.1 server\nhostname-only\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-08",
    "category": "Grep & Regex",
    "title": "Case-Insensitive Recursive Search Ignoring Binaries",
    "difficulty": "Intermediate",
    "description": "Search directory configs recursively (-r), ignoring binaries (-I), ignoring case (-i) for 'API_KEY'.",
    "objective": "grep -rIi 'API_KEY' configs",
    "hints": [
      "Command: grep -rIi 'API_KEY' configs"
    ],
    "solution": "grep -rIi 'API_KEY' configs",
    "accepted_regex": [
      "grep\\s+(-rIi|-riI|-Iri|-Iir)\\s+['\\\"]?API_KEY['\\\"]?\\s+configs"
    ],
    "setup_files": {
      "configs/settings.json": "{\"api_key\": \"123\"}\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-09",
    "category": "Grep & Regex",
    "title": "List Files NOT Matching Pattern with grep -L",
    "difficulty": "Intermediate",
    "description": "Print filenames of *.conf that do NOT contain 'PermitRootLogin'.",
    "objective": "grep -L 'PermitRootLogin' *.conf",
    "hints": [
      "Command: grep -L 'PermitRootLogin' *.conf"
    ],
    "solution": "grep -L 'PermitRootLogin' *.conf",
    "accepted_regex": [
      "grep\\s+-L\\s+['\\\"]?PermitRootLogin['\\\"]?\\s+\\*\\.conf"
    ],
    "setup_files": {
      "sec.conf": "PermitRootLogin no\n",
      "insec.conf": "Port 22\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-10",
    "category": "Grep & Regex",
    "title": "Extract Only the Matching String with grep -o",
    "difficulty": "Intermediate",
    "description": "Extract only the IP address part from connection.log using grep -o.",
    "objective": "grep -o -E '[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+' connection.log",
    "hints": [
      "Command: grep -o -E '[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+' connection.log"
    ],
    "solution": "grep -o -E '[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+' connection.log",
    "accepted_regex": [
      "grep\\s+(-oE|-Eo|-o\\s+-E|-E\\s+-o)\\s+['\\\"][0-9]"
    ],
    "setup_files": {
      "connection.log": "Connected from 10.0.0.5 port 22\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-11",
    "category": "Grep & Regex",
    "title": "Quiet Match for Script Exit Code Check",
    "difficulty": "Easy",
    "description": "Check if 'ansible' exists in /etc/passwd quietly without outputting text.",
    "objective": "grep -q 'ansible' /etc/passwd",
    "hints": [
      "Command: grep -q 'ansible' /etc/passwd"
    ],
    "solution": "grep -q 'ansible' /etc/passwd",
    "accepted_regex": [
      "grep\\s+-q\\s+['\\\"]?ansible['\\\"]?\\s+\\/etc\\/passwd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-12",
    "category": "Grep & Regex",
    "title": "Match Lines Ending with Semicolon",
    "difficulty": "Easy",
    "description": "Search code.c for lines ending with a semicolon (;).",
    "objective": "grep ';$' code.c",
    "hints": [
      "Command: grep ';$' code.c"
    ],
    "solution": "grep ';$' code.c",
    "accepted_regex": [
      "grep\\s+['\\\"]?;\\$['\\\"]?\\s+code\\.c"
    ],
    "setup_files": {
      "code.c": "int x = 10;\nif (x) {\n    return 0;\n}\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-13",
    "category": "Grep & Regex",
    "title": "Match Lines with Alternative Patterns using -e",
    "difficulty": "Intermediate",
    "description": "Search syslog for either 'WARNING' or 'CRITICAL' using multiple -e flags.",
    "objective": "grep -e 'WARNING' -e 'CRITICAL' syslog",
    "hints": [
      "Command: grep -e 'WARNING' -e 'CRITICAL' syslog"
    ],
    "solution": "grep -e 'WARNING' -e 'CRITICAL' syslog",
    "accepted_regex": [
      "grep\\s+-e\\s+['\\\"]?WARNING['\\\"]?\\s+-e\\s+['\\\"]?CRITICAL['\\\"]?\\s+syslog",
      "grep\\s+-E\\s+['\\\"](WARNING|CRITICAL)"
    ],
    "setup_files": {
      "syslog": "INFO: ok\nWARNING: disk\nCRITICAL: cpu\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-14",
    "category": "Grep & Regex",
    "title": "Match Exact Entire Line with grep -x",
    "difficulty": "Intermediate",
    "description": "Find lines in state.txt that match exactly 'ENABLED' with nothing else on the line.",
    "objective": "grep -x 'ENABLED' state.txt",
    "hints": [
      "Command: grep -x 'ENABLED' state.txt"
    ],
    "solution": "grep -x 'ENABLED' state.txt",
    "accepted_regex": [
      "grep\\s+-x\\s+['\\\"]?ENABLED['\\\"]?\\s+state\\.txt"
    ],
    "setup_files": {
      "state.txt": "ENABLED\nENABLED_SOON\nPRE_ENABLED\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-15",
    "category": "Grep & Regex",
    "title": "Match Hexadecimal MAC Addresses",
    "difficulty": "Advanced",
    "description": "Search ifcfg.txt for standard MAC addresses matching format XX:XX:XX:XX:XX:XX.",
    "objective": "grep -E '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' ifcfg.txt",
    "hints": [
      "Command: grep -E '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' ifcfg.txt"
    ],
    "solution": "grep -E '([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}' ifcfg.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+.*\\[0-9A-Fa-f\\]\\{2\\}:"
    ],
    "setup_files": {
      "ifcfg.txt": "HWADDR=52:54:00:12:34:56\nNAME=eth0\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-16",
    "category": "Grep & Regex",
    "title": "Invert Match Excluding Multiple Status Codes",
    "difficulty": "Intermediate",
    "description": "Read web.log. Exclude all lines containing status code 200 or 304.",
    "objective": "grep -v -E ' (200|304) ' web.log",
    "hints": [
      "Command: grep -v -E ' (200|304) ' web.log"
    ],
    "solution": "grep -v -E ' (200|304) ' web.log",
    "accepted_regex": [
      "grep\\s+(-vE|-Ev|-E\\s+-v|-v\\s+-E)\\s+.*(200\\|304)"
    ],
    "setup_files": {
      "web.log": "GET / 200 120\nGET /img 304 0\nPOST /err 500 40\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-17",
    "category": "Grep & Regex",
    "title": "Count Blank Lines in Script",
    "difficulty": "Easy",
    "description": "Count how many completely empty lines exist in script.sh.",
    "objective": "grep -c '^$' script.sh",
    "hints": [
      "Command: grep -c '^$' script.sh"
    ],
    "solution": "grep -c '^$' script.sh",
    "accepted_regex": [
      "grep\\s+-c\\s+['\\\"]?\\^\\$['\\\"]?\\s+script\\.sh"
    ],
    "setup_files": {
      "script.sh": "#!/bin/bash\n\necho 1\n\necho 2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-18",
    "category": "Grep & Regex",
    "title": "Match Standard UUID Format Lines",
    "difficulty": "Advanced",
    "description": "Find lines containing a standard 36-character UUID with 8-4-4-4-12 hex format.",
    "objective": "grep -E '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' fstab.test",
    "hints": [
      "Command: grep -E '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' fstab.test"
    ],
    "solution": "grep -E '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' fstab.test",
    "accepted_regex": [
      "grep\\s+-E\\s+.*\\[0-9a-f\\]\\{8\\}-"
    ],
    "setup_files": {
      "fstab.test": "UUID=c1b9d5a3-0000-4b21-8899-abcdef123456 / xfs defaults 0 0\n/dev/sda1 /boot ext4 defaults 0 0\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-19",
    "category": "Grep & Regex",
    "title": "Print 3 Lines After Each Matching Error",
    "difficulty": "Easy",
    "description": "Search trace.log for 'Exception' printing 3 lines after each match (-A 3).",
    "objective": "grep -A 3 'Exception' trace.log",
    "hints": [
      "Command: grep -A 3 'Exception' trace.log"
    ],
    "solution": "grep -A 3 'Exception' trace.log",
    "accepted_regex": [
      "grep\\s+-A\\s*3\\s+['\\\"]?Exception['\\\"]?\\s+trace\\.log"
    ],
    "setup_files": {
      "trace.log": "Exception in thread main\nat class.method\nat class.caller\nat main\ninfo\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-20",
    "category": "Grep & Regex",
    "title": "Print 4 Lines Before Each Matching Error",
    "difficulty": "Easy",
    "description": "Search trace.log for 'CRASH' printing 4 lines before each match (-B 4).",
    "objective": "grep -B 4 'CRASH' trace.log",
    "hints": [
      "Command: grep -B 4 'CRASH' trace.log"
    ],
    "solution": "grep -B 4 'CRASH' trace.log",
    "accepted_regex": [
      "grep\\s+-B\\s*4\\s+['\\\"]?CRASH['\\\"]?\\s+trace\\.log"
    ],
    "setup_files": {
      "trace.log": "step1\nstep2\nstep3\nstep4\nCRASH\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-21",
    "category": "Grep & Regex",
    "title": "Filter Directives in /etc/fstab Starting with UUID",
    "difficulty": "Easy",
    "description": "Search fstab for active lines beginning with 'UUID='.",
    "objective": "grep '^UUID=' /etc/fstab",
    "hints": [
      "Command: grep '^UUID=' /etc/fstab"
    ],
    "solution": "grep '^UUID=' /etc/fstab",
    "accepted_regex": [
      "grep\\s+['\\\"]?\\^UUID=['\\\"]?\\s+\\/etc\\/fstab"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-22",
    "category": "Grep & Regex",
    "title": "Match ISO-8601 Date Timestamps",
    "difficulty": "Intermediate",
    "description": "Search events.log for lines starting with dates formatted as YYYY-MM-DD.",
    "objective": "grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log",
    "hints": [
      "Command: grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log"
    ],
    "solution": "grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]\\^\\[0-9\\]\\{4\\}-\\[0-9\\]\\{2\\}-\\[0-9\\]\\{2\\}['\\\"]"
    ],
    "setup_files": {
      "events.log": "2026-10-09 12:00:00 Started\ninvalid timestamp\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-23",
    "category": "Grep & Regex",
    "title": "Match Port Digits in Config Lines",
    "difficulty": "Intermediate",
    "description": "Search ports.conf for lines containing 'Port' followed by 2 to 5 digits.",
    "objective": "grep -E 'Port [0-9]{2,5}' ports.conf",
    "hints": [
      "Command: grep -E 'Port [0-9]{2,5}' ports.conf"
    ],
    "solution": "grep -E 'Port [0-9]{2,5}' ports.conf",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]Port\\s+\\[0-9\\]\\{2,5\\}['\\\"]"
    ],
    "setup_files": {
      "ports.conf": "Port 22\nPort 8080\nPort abc\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-24",
    "category": "Grep & Regex",
    "title": "Extract Email Addresses Using Regex",
    "difficulty": "Advanced",
    "description": "Extract valid email addresses from contacts.txt using grep -o -E.",
    "objective": "grep -o -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt",
    "hints": [
      "Command: grep -o -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt"
    ],
    "solution": "grep -o -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt",
    "accepted_regex": [
      "grep\\s+(-oE|-Eo|-o\\s+-E|-E\\s+-o)\\s+.*@.*contacts\\.txt"
    ],
    "setup_files": {
      "contacts.txt": "Contact admin@example.com or support@redhat.com\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-25",
    "category": "Grep & Regex",
    "title": "Invert Matching Specific User Logins",
    "difficulty": "Easy",
    "description": "Filter out lines from users.log containing 'admin' or 'root'.",
    "objective": "grep -v -E '(admin|root)' users.log",
    "hints": [
      "Command: grep -v -E '(admin|root)' users.log"
    ],
    "solution": "grep -v -E '(admin|root)' users.log",
    "accepted_regex": [
      "grep\\s+(-vE|-Ev|-E\\s+-v|-v\\s+-E)\\s+.*(admin\\|root)"
    ],
    "setup_files": {
      "users.log": "user: admin\nuser: alice\nuser: root\nuser: bob\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-26",
    "category": "Grep & Regex",
    "title": "Display Matching Line Numbers for TODO Items",
    "difficulty": "Easy",
    "description": "Search code.py for 'TODO' displaying each match with its line number.",
    "objective": "grep -n 'TODO' code.py",
    "hints": [
      "Command: grep -n 'TODO' code.py"
    ],
    "solution": "grep -n 'TODO' code.py",
    "accepted_regex": [
      "grep\\s+-n\\s+['\\\"]?TODO['\\\"]?\\s+code\\.py"
    ],
    "setup_files": {
      "code.py": "# TODO: fix bug\ndef run():\n    pass\n# TODO: test\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-27",
    "category": "Grep & Regex",
    "title": "Search Only Compressed Gz Logs with zgrep",
    "difficulty": "Intermediate",
    "description": "Search compressed file access.log.gz directly for '404' using zgrep.",
    "objective": "zgrep '404' access.log.gz",
    "hints": [
      "Command: zgrep '404' access.log.gz"
    ],
    "solution": "zgrep '404' access.log.gz",
    "accepted_regex": [
      "zgrep\\s+['\\\"]?404['\\\"]?\\s+access\\.log\\.gz"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-28",
    "category": "Grep & Regex",
    "title": "Match Digits Only Lines",
    "difficulty": "Easy",
    "description": "Search numbers.txt for lines composed exclusively of numeric digits.",
    "objective": "grep -E '^[0-9]+$' numbers.txt",
    "hints": [
      "Command: grep -E '^[0-9]+$' numbers.txt"
    ],
    "solution": "grep -E '^[0-9]+$' numbers.txt",
    "accepted_regex": [
      "grep\\s+(-E\\s+)?['\\\"]\\^\\[0-9\\]\\+\\$['\\\"]\\s+numbers\\.txt"
    ],
    "setup_files": {
      "numbers.txt": "12345\n12a34\n999\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-29",
    "category": "Grep & Regex",
    "title": "Search Case-Insensitively for Warning or Error",
    "difficulty": "Easy",
    "description": "Search app.log case-insensitively for 'error'.",
    "objective": "grep -i 'error' app.log",
    "hints": [
      "Command: grep -i 'error' app.log"
    ],
    "solution": "grep -i 'error' app.log",
    "accepted_regex": [
      "grep\\s+-i\\s+['\\\"]?error['\\\"]?\\s+app\\.log"
    ],
    "setup_files": {
      "app.log": "ERROR: fatal\nInfo: ok\nError: minor\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-30",
    "category": "Grep & Regex",
    "title": "Match Beginning and End of Word for Sysctl",
    "difficulty": "Intermediate",
    "description": "Search sysctl.conf for directives starting with 'net.ipv4.ip_forward'.",
    "objective": "grep '^net\\.ipv4\\.ip_forward' sysctl.conf",
    "hints": [
      "Command: grep '^net\\.ipv4\\.ip_forward' sysctl.conf"
    ],
    "solution": "grep '^net\\.ipv4\\.ip_forward' sysctl.conf",
    "accepted_regex": [
      "grep\\s+['\\\"]\\^net\\\\\\.ipv4\\\\\\.ip_forward['\\\"]\\s+sysctl\\.conf"
    ],
    "setup_files": {
      "sysctl.conf": "net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.disable_ipv6 = 1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-01",
    "category": "Sed Stream Editing",
    "title": "SELinux Enforce Substitution",
    "difficulty": "Easy",
    "description": "Substitute 'SELINUX=enforcing' with 'SELINUX=permissive' in selinux.conf.",
    "objective": "sed 's/enforcing/permissive/' selinux.conf",
    "hints": [
      "Command: sed 's/enforcing/permissive/' selinux.conf"
    ],
    "solution": "sed 's/enforcing/permissive/' selinux.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/enforcing\\/permissive\\/['\\\"]\\s+selinux\\.conf"
    ],
    "setup_files": {
      "selinux.conf": "SELINUX=enforcing\nSELINUXTYPE=targeted\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-02",
    "category": "Sed Stream Editing",
    "title": "Delete Comment Lines Using Pattern Addressing",
    "difficulty": "Intermediate",
    "description": "Delete all lines beginning with '#' from crontab.test.",
    "objective": "sed '/^#/d' crontab.test",
    "hints": [
      "Command: sed '/^#/d' crontab.test"
    ],
    "solution": "sed '/^#/d' crontab.test",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/\\^\\#\\/d['\\\"]\\s+crontab\\.test"
    ],
    "setup_files": {
      "crontab.test": "# backup\n0 2 * * * root /bin/backup\n# audit\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-03",
    "category": "Sed Stream Editing",
    "title": "Replace Directory Path with Custom Delimiter",
    "difficulty": "Intermediate",
    "description": "Replace '/var/www/html' with '/data/www/html' in vhost.conf using '#' delimiter.",
    "objective": "sed 's#/var/www/html#/data/www/html#g' vhost.conf",
    "hints": [
      "Command: sed 's#/var/www/html#/data/www/html#g' vhost.conf"
    ],
    "solution": "sed 's#/var/www/html#/data/www/html#g' vhost.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]s[#|@]\\/var\\/www\\/html[#|@]\\/data\\/www\\/html[#|@](g)?['\\\"]\\s+vhost\\.conf"
    ],
    "setup_files": {
      "vhost.conf": "DocumentRoot \"/var/www/html\"\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-04",
    "category": "Sed Stream Editing",
    "title": "Print Specific Line Range with sed -n",
    "difficulty": "Intermediate",
    "description": "Print lines 3 through 5 (inclusive) from audit.log.",
    "objective": "sed -n '3,5p' audit.log",
    "hints": [
      "Command: sed -n '3,5p' audit.log"
    ],
    "solution": "sed -n '3,5p' audit.log",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]3,5p['\\\"]\\s+audit\\.log"
    ],
    "setup_files": {
      "audit.log": "1\n2\n3\n4\n5\n6\n7\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-05",
    "category": "Sed Stream Editing",
    "title": "Strip Leading Whitespace Indentation",
    "difficulty": "Intermediate",
    "description": "Remove leading spaces and tabs from code.txt.",
    "objective": "sed 's/^[ \t]*//' code.txt",
    "hints": [
      "Command: sed 's/^[ \t]*//' code.txt"
    ],
    "solution": "sed 's/^[ \t]*//' code.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\^\\[\\s*\\\\t\\]\\*\\/.*['\\\"]\\s+code\\.txt"
    ],
    "setup_files": {
      "code.txt": "    hello\n        world\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-06",
    "category": "Sed Stream Editing",
    "title": "Delete All Empty Blank Lines with sed",
    "difficulty": "Easy",
    "description": "Delete all empty lines (zero characters) from notes.txt.",
    "objective": "sed '/^$/d' notes.txt",
    "hints": [
      "Command: sed '/^$/d' notes.txt"
    ],
    "solution": "sed '/^$/d' notes.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/\\^\\$\\/d['\\\"]\\s+notes\\.txt"
    ],
    "setup_files": {
      "notes.txt": "a\n\nb\n\nc\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-07",
    "category": "Sed Stream Editing",
    "title": "Strip Windows Carriage Returns",
    "difficulty": "Intermediate",
    "description": "Remove trailing DOS/Windows \\r carriage returns from windows.txt.",
    "objective": "sed 's/\\r$//' windows.txt",
    "hints": [
      "Command: sed 's/\\r$//' windows.txt"
    ],
    "solution": "sed 's/\\r$//' windows.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\\\r\\$\\/\\/['\\\"]\\s+windows\\.txt"
    ],
    "setup_files": {
      "windows.txt": "line1\r\nline2\r\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-08",
    "category": "Sed Stream Editing",
    "title": "Global Replace All Occurrences on Lines",
    "difficulty": "Easy",
    "description": "Replace every occurrence of 'foo' with 'bar' globally across words.txt.",
    "objective": "sed 's/foo/bar/g' words.txt",
    "hints": [
      "Command: sed 's/foo/bar/g' words.txt"
    ],
    "solution": "sed 's/foo/bar/g' words.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/foo\\/bar\\/g['\\\"]\\s+words\\.txt"
    ],
    "setup_files": {
      "words.txt": "foo and foo are foos\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-09",
    "category": "Sed Stream Editing",
    "title": "Delete Specific First Line of File",
    "difficulty": "Easy",
    "description": "Delete the first header line (line 1) from data.csv.",
    "objective": "sed '1d' data.csv",
    "hints": [
      "Command: sed '1d' data.csv"
    ],
    "solution": "sed '1d' data.csv",
    "accepted_regex": [
      "sed\\s+['\\\"]1d['\\\"]\\s+data\\.csv"
    ],
    "setup_files": {
      "data.csv": "id,name\n1,alice\n2,bob\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-10",
    "category": "Sed Stream Editing",
    "title": "Delete the Last Line of File",
    "difficulty": "Easy",
    "description": "Delete the final line ($d) from records.txt.",
    "objective": "sed '$d' records.txt",
    "hints": [
      "Command: sed '$d' records.txt"
    ],
    "solution": "sed '$d' records.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\$d['\\\"]\\s+records\\.txt"
    ],
    "setup_files": {
      "records.txt": "first\nsecond\ntrailer\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-11",
    "category": "Sed Stream Editing",
    "title": "Replace Second Occurrence on Each Line",
    "difficulty": "Intermediate",
    "description": "Replace only the 2nd occurrence of 'test' on each line with 'pass'.",
    "objective": "sed 's/test/pass/2' items.txt",
    "hints": [
      "Command: sed 's/test/pass/2' items.txt"
    ],
    "solution": "sed 's/test/pass/2' items.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/test\\/pass\\/2['\\\"]\\s+items\\.txt"
    ],
    "setup_files": {
      "items.txt": "test one test two test three\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-12",
    "category": "Sed Stream Editing",
    "title": "Comment Out Matching Configuration Directives",
    "difficulty": "Intermediate",
    "description": "Prepend '#' to lines starting with 'Listen 80' in httpd.conf.",
    "objective": "sed 's/^Listen 80/#Listen 80/' httpd.conf",
    "hints": [
      "Command: sed 's/^Listen 80/#Listen 80/' httpd.conf"
    ],
    "solution": "sed 's/^Listen 80/#Listen 80/' httpd.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\^Listen\\s+80\\/#Listen\\s+80\\/['\\\"]\\s+httpd\\.conf"
    ],
    "setup_files": {
      "httpd.conf": "Listen 80\nServerName test\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-13",
    "category": "Sed Stream Editing",
    "title": "Extract Block Between Two Delimiter Tags",
    "difficulty": "Advanced",
    "description": "Print content exclusively between lines matching 'START' and 'END'.",
    "objective": "sed -n '/START/,/END/p' block.txt",
    "hints": [
      "Command: sed -n '/START/,/END/p' block.txt"
    ],
    "solution": "sed -n '/START/,/END/p' block.txt",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]\\/START\\/,\\/END\\/p['\\\"]\\s+block\\.txt"
    ],
    "setup_files": {
      "block.txt": "preface\nSTART\ninside1\ninside2\nEND\npost\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-14",
    "category": "Sed Stream Editing",
    "title": "Strip Trailing Whitespace at End of Lines",
    "difficulty": "Intermediate",
    "description": "Remove all spaces and tabs at line ends from code.py.",
    "objective": "sed 's/[ \t]*$//' code.py",
    "hints": [
      "Command: sed 's/[ \t]*$//' code.py"
    ],
    "solution": "sed 's/[ \t]*$//' code.py",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\[\\s*\\\\t\\]\\*\\$\\/\\/['\\\"]\\s+code\\.py"
    ],
    "setup_files": {
      "code.py": "def test():   \n    return 1 \n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-15",
    "category": "Sed Stream Editing",
    "title": "Replace Only on Lines Matching Condition",
    "difficulty": "Intermediate",
    "description": "On lines containing 'port', substitute '80' with '8080'.",
    "objective": "sed '/port/s/80/8080/' app.conf",
    "hints": [
      "Command: sed '/port/s/80/8080/' app.conf"
    ],
    "solution": "sed '/port/s/80/8080/' app.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/port\\/s\\/80\\/8080\\/['\\\"]\\s+app\\.conf"
    ],
    "setup_files": {
      "app.conf": "port = 80\nmax_clients = 80\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-16",
    "category": "Sed Stream Editing",
    "title": "In-Place File Edit with Backup Creation",
    "difficulty": "Intermediate",
    "description": "Edit config.env in-place creating config.env.bak, replacing 'DEV' with 'PROD'.",
    "objective": "sed -i.bak 's/DEV/PROD/' config.env",
    "hints": [
      "Command: sed -i.bak 's/DEV/PROD/' config.env"
    ],
    "solution": "sed -i.bak 's/DEV/PROD/' config.env",
    "accepted_regex": [
      "sed\\s+-i\\.bak\\s+['\\\"]s\\/DEV\\/PROD\\/['\\\"]\\s+config\\.env"
    ],
    "setup_files": {
      "config.env": "ENV=DEV\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-17",
    "category": "Sed Stream Editing",
    "title": "Delete Lines 2 Through 4",
    "difficulty": "Easy",
    "description": "Delete lines 2 through 4 from numbers.txt.",
    "objective": "sed '2,4d' numbers.txt",
    "hints": [
      "Command: sed '2,4d' numbers.txt"
    ],
    "solution": "sed '2,4d' numbers.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]2,4d['\\\"]\\s+numbers\\.txt"
    ],
    "setup_files": {
      "numbers.txt": "1\n2\n3\n4\n5\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-18",
    "category": "Sed Stream Editing",
    "title": "Transliterate Lowercase Vowels to Uppercase",
    "difficulty": "Intermediate",
    "description": "Translate characters 'aeiou' to 'AEIOU' in vowels.txt using sed y command.",
    "objective": "sed 'y/aeiou/AEIOU/' vowels.txt",
    "hints": [
      "Command: sed 'y/aeiou/AEIOU/' vowels.txt"
    ],
    "solution": "sed 'y/aeiou/AEIOU/' vowels.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]y\\/aeiou\\/AEIOU\\/['\\\"]\\s+vowels\\.txt"
    ],
    "setup_files": {
      "vowels.txt": "quick brown fox\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-19",
    "category": "Sed Stream Editing",
    "title": "Append Line After Match with 'a' Command",
    "difficulty": "Intermediate",
    "description": "Append a new line 'Include conf.d/*.conf' after line matching 'ServerRoot' in httpd.conf.",
    "objective": "sed '/ServerRoot/a Include conf.d/*.conf' httpd.conf",
    "hints": [
      "Command: sed '/ServerRoot/a Include conf.d/*.conf' httpd.conf"
    ],
    "solution": "sed '/ServerRoot/a Include conf.d/*.conf' httpd.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/ServerRoot\\/a\\s+Include\\s+conf\\.d\\/\\*\\.conf['\\\"]\\s+httpd\\.conf"
    ],
    "setup_files": {
      "httpd.conf": "ServerRoot /etc/httpd\nListen 80\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-20",
    "category": "Sed Stream Editing",
    "title": "Insert Line Before Match with 'i' Command",
    "difficulty": "Intermediate",
    "description": "Insert '# Managed by Ansible' before line 1 in hosts.txt.",
    "objective": "sed '1i # Managed by Ansible' hosts.txt",
    "hints": [
      "Command: sed '1i # Managed by Ansible' hosts.txt"
    ],
    "solution": "sed '1i # Managed by Ansible' hosts.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]1i\\s+# Managed by Ansible['\\\"]\\s+hosts\\.txt"
    ],
    "setup_files": {
      "hosts.txt": "127.0.0.1 localhost\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-21",
    "category": "Sed Stream Editing",
    "title": "Double Space Text by Appending Newlines",
    "difficulty": "Intermediate",
    "description": "Insert an empty line after every line using sed G command.",
    "objective": "sed 'G' file.txt",
    "hints": [
      "Command: sed 'G' file.txt"
    ],
    "solution": "sed 'G' file.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]G['\\\"]\\s+file\\.txt"
    ],
    "setup_files": {
      "file.txt": "line1\nline2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-22",
    "category": "Sed Stream Editing",
    "title": "Delete Lines NOT Matching Pattern",
    "difficulty": "Intermediate",
    "description": "Delete all lines except those matching 'KEEP' in keep.txt.",
    "objective": "sed '/KEEP/!d' keep.txt",
    "hints": [
      "Command: sed '/KEEP/!d' keep.txt"
    ],
    "solution": "sed '/KEEP/!d' keep.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/KEEP\\/!d['\\\"]\\s+keep\\.txt"
    ],
    "setup_files": {
      "keep.txt": "drop1\nKEEP this\ndrop2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-23",
    "category": "Sed Stream Editing",
    "title": "Replace HTML Tags with Nothing",
    "difficulty": "Advanced",
    "description": "Strip all basic HTML tags (<...>) from page.html.",
    "objective": "sed 's/<[^>]*>//g' page.html",
    "hints": [
      "Command: sed 's/<[^>]*>//g' page.html"
    ],
    "solution": "sed 's/<[^>]*>//g' page.html",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/<\\[\\^>\\]\\*>\\/\\/g['\\\"]\\s+page\\.html"
    ],
    "setup_files": {
      "page.html": "<h1>Title</h1><p>Text</p>\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-24",
    "category": "Sed Stream Editing",
    "title": "Print Only the First Line and Quit",
    "difficulty": "Easy",
    "description": "Print line 1 and immediately quit using sed '1q'.",
    "objective": "sed '1q' big.log",
    "hints": [
      "Command: sed '1q' big.log"
    ],
    "solution": "sed '1q' big.log",
    "accepted_regex": [
      "sed\\s+['\\\"]1q['\\\"]\\s+big\\.log"
    ],
    "setup_files": {
      "big.log": "first\nsecond\nthird\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-25",
    "category": "Sed Stream Editing",
    "title": "Substitute Port at the End of Line",
    "difficulty": "Easy",
    "description": "Change ':80' to ':443' at the end of the line in endpoint.txt.",
    "objective": "sed 's/:80$/:443/' endpoint.txt",
    "hints": [
      "Command: sed 's/:80$/:443/' endpoint.txt"
    ],
    "solution": "sed 's/:80$/:443/' endpoint.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/:80\\$\\/:443\\/['\\\"]\\s+endpoint\\.txt"
    ],
    "setup_files": {
      "endpoint.txt": "http://example.com:80\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-01",
    "category": "Awk Processing",
    "title": "Extract System Users with Bash Shell",
    "difficulty": "Intermediate",
    "description": "Using delimiter ':', print $1 and $3 as 'user:UID' for accounts with shell '/bin/bash'.",
    "objective": "awk -F':' '$7 == \"/bin/bash\" {print $1\":\"$3}' passwd.sample",
    "hints": [
      "Command: awk -F':' '$7 == \"/bin/bash\" {print $1\":\"$3}' passwd.sample"
    ],
    "solution": "awk -F':' '$7 == \"/bin/bash\" {print $1\":\"$3}' passwd.sample",
    "accepted_regex": [
      "awk\\s+-F['\\\"]?:['\\\"]?\\s+['\\\"]\\s*\\$7\\s*==\\s*[\\\"']/bin/bash[\\\"'].*passwd\\.sample"
    ],
    "setup_files": {
      "passwd.sample": "root:x:0:0:root:/root:/bin/bash\nbin:x:1:1:bin:/bin:/sbin/nologin\nansible:x:1001:1001::/home/ansible:/bin/bash\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-02",
    "category": "Awk Processing",
    "title": "Sum Column Values of Log Transfer Bytes",
    "difficulty": "Intermediate",
    "description": "Calculate and print total sum of transferred byte sizes in column 2 of transfers.tsv.",
    "objective": "awk '{sum += $2} END {print sum}' transfers.tsv",
    "hints": [
      "Command: awk '{sum += $2} END {print sum}' transfers.tsv"
    ],
    "solution": "awk '{sum += $2} END {print sum}' transfers.tsv",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{sum\\s*\\+=\\s*\\$2\\}\\s*END\\s*\\{\\s*print\\s+sum\\s*\\}['\\\"]\\s+transfers\\.tsv"
    ],
    "setup_files": {
      "transfers.tsv": "fileA 100\nfileB 250\nfileC 50\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-03",
    "category": "Awk Processing",
    "title": "Print Last Field of Variable Length Log Lines",
    "difficulty": "Intermediate",
    "description": "Print only the final field ($NF) of each line in commands.log.",
    "objective": "awk '{print $NF}' commands.log",
    "hints": [
      "Command: awk '{print $NF}' commands.log"
    ],
    "solution": "awk '{print $NF}' commands.log",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{print\\s+\\$NF\\}['\\\"]\\s+commands\\.log"
    ],
    "setup_files": {
      "commands.log": "cmd1 arg1\ncmd2 arg1 arg2 end\ncmd3 done\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-04",
    "category": "Awk Processing",
    "title": "Filter Human Interactive Users with UID >= 1000",
    "difficulty": "Intermediate",
    "description": "Print username from passwd.sample for users with UID >= 1000 and < 65534.",
    "objective": "awk -F':' '$3 >= 1000 && $3 < 65534 {print $1}' passwd.sample",
    "hints": [
      "Command: awk -F':' '$3 >= 1000 && $3 < 65534 {print $1}' passwd.sample"
    ],
    "solution": "awk -F':' '$3 >= 1000 && $3 < 65534 {print $1}' passwd.sample",
    "accepted_regex": [
      "awk\\s+-F['\\\"]?:['\\\"]?\\s+['\\\"]\\s*\\$3\\s*>=\\s*1000"
    ],
    "setup_files": {
      "passwd.sample": "root:x:0:0:::/bin/bash\nuser1:x:1001:1001:::/bin/bash\nnobody:x:65534:65534:::/sbin/nologin\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-05",
    "category": "Awk Processing",
    "title": "Custom Output Field Separator with OFS",
    "difficulty": "Intermediate",
    "description": "Print col 1 and 2 separated by ' -> ' from tab-delimited users.tsv.",
    "objective": "awk -F'\\t' 'BEGIN {OFS=\" -> \"} {print $1, $2}' users.tsv",
    "hints": [
      "Command: awk -F'\\t' 'BEGIN {OFS=\" -> \"} {print $1, $2}' users.tsv"
    ],
    "solution": "awk -F'\\t' 'BEGIN {OFS=\" -> \"} {print $1, $2}' users.tsv",
    "accepted_regex": [
      "awk\\s+.*BEGIN\\s*\\{\\s*OFS\\s*=\\s*[\"\\']\\s*->\\s*[\"\\']\\s*\\}.*users\\.tsv"
    ],
    "setup_files": {
      "users.tsv": "alice\tadmin\nbob\tuser\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-06",
    "category": "Awk Processing",
    "title": "Print Total Number of Records in File",
    "difficulty": "Easy",
    "description": "Use awk's built-in NR variable in END block to print total record count of items.txt.",
    "objective": "awk 'END {print NR}' items.txt",
    "hints": [
      "Command: awk 'END {print NR}' items.txt"
    ],
    "solution": "awk 'END {print NR}' items.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]END\\s*\\{\\s*print\\s+NR\\s*\\}['\\\"]\\s+items\\.txt"
    ],
    "setup_files": {
      "items.txt": "a\nb\nc\nd\ne\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-07",
    "category": "Awk Processing",
    "title": "Compute Average Value of Column 2",
    "difficulty": "Intermediate",
    "description": "Calculate and print average of values in column 2 of scores.txt.",
    "objective": "awk '{sum += $2} END {print sum/NR}' scores.txt",
    "hints": [
      "Command: awk '{sum += $2} END {print sum/NR}' scores.txt"
    ],
    "solution": "awk '{sum += $2} END {print sum/NR}' scores.txt",
    "accepted_regex": [
      "awk\\s+['\\\"].*sum\\s*\\+=\\s*\\$2.*print\\s+sum\\/NR.*scores\\.txt"
    ],
    "setup_files": {
      "scores.txt": "test1 80\ntest2 100\ntest3 90\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-08",
    "category": "Awk Processing",
    "title": "Filter Lines Having More Than 4 Fields",
    "difficulty": "Easy",
    "description": "Print lines from table.txt where Number of Fields (NF) is greater than 4.",
    "objective": "awk 'NF > 4' table.txt",
    "hints": [
      "Command: awk 'NF > 4' table.txt"
    ],
    "solution": "awk 'NF > 4' table.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]NF\\s*>\\s*4['\\\"]\\s+table\\.txt"
    ],
    "setup_files": {
      "table.txt": "a b c\na b c d e\na b\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-09",
    "category": "Awk Processing",
    "title": "Count Frequency of IP Addresses with Array",
    "difficulty": "Advanced",
    "description": "Count and print frequency of IPs in column 1 of access.log using associative arrays.",
    "objective": "awk '{ips[$1]++} END {for (ip in ips) print ips[ip], ip}' access.log",
    "hints": [
      "Command: awk '{ips[$1]++} END {for (ip in ips) print ips[ip], ip}' access.log"
    ],
    "solution": "awk '{ips[$1]++} END {for (ip in ips) print ips[ip], ip}' access.log",
    "accepted_regex": [
      "awk\\s+['\\\"].*ips\\[\\$1\\]\\+\\+.*for\\s*\\(.*access\\.log"
    ],
    "setup_files": {
      "access.log": "10.0.0.1 - /index\n10.0.0.2 - /login\n10.0.0.1 - /img\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-10",
    "category": "Awk Processing",
    "title": "Filter Rows Where Column 3 Matches Regex",
    "difficulty": "Intermediate",
    "description": "Print rows in services.txt where column 3 starts with 'HTTP'.",
    "objective": "awk '$3 ~ /^HTTP/' services.txt",
    "hints": [
      "Command: awk '$3 ~ /^HTTP/' services.txt"
    ],
    "solution": "awk '$3 ~ /^HTTP/' services.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\s*\\$3\\s*~\\s*\\/(\\^)?HTTP\\/['\\\"]\\s+services\\.txt"
    ],
    "setup_files": {
      "services.txt": "1 web HTTPS\n2 mail SMTP\n3 web HTTP/1.1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-11",
    "category": "Awk Processing",
    "title": "Format Output with printf Columns",
    "difficulty": "Intermediate",
    "description": "Print formatted columns '%-10s %5d\\n' for $1 and $2 from inventory.txt.",
    "objective": "awk '{printf \"%-10s %5d\\n\", $1, $2}' inventory.txt",
    "hints": [
      "Command: awk '{printf \"%-10s %5d\\n\", $1, $2}' inventory.txt"
    ],
    "solution": "awk '{printf \"%-10s %5d\\n\", $1, $2}' inventory.txt",
    "accepted_regex": [
      "awk\\s+['\\\"].*printf.*%-10s.*inventory\\.txt"
    ],
    "setup_files": {
      "inventory.txt": "apples 50\noranges 120\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-12",
    "category": "Awk Processing",
    "title": "Print Line Number Alongside Each Record",
    "difficulty": "Easy",
    "description": "Print record number NR followed by the full line $0 from sample.txt.",
    "objective": "awk '{print NR, $0}' sample.txt",
    "hints": [
      "Command: awk '{print NR, $0}' sample.txt"
    ],
    "solution": "awk '{print NR, $0}' sample.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{print\\s+NR,\\s*\\$0\\}['\\\"]\\s+sample\\.txt"
    ],
    "setup_files": {
      "sample.txt": "first\nsecond\nthird\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-13",
    "category": "Awk Processing",
    "title": "Print Lines Exceeding 80 Characters in Length",
    "difficulty": "Intermediate",
    "description": "Find and print lines in code.py whose length exceeds 80 characters.",
    "objective": "awk 'length($0) > 80' code.py",
    "hints": [
      "Command: awk 'length($0) > 80' code.py"
    ],
    "solution": "awk 'length($0) > 80' code.py",
    "accepted_regex": [
      "awk\\s+['\\\"]length\\((\\$0)?\\)\\s*>\\s*80['\\\"]\\s+code\\.py"
    ],
    "setup_files": {
      "code.py": "short line\nxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-14",
    "category": "Awk Processing",
    "title": "Preserve Header Line While Filtering Rows",
    "difficulty": "Intermediate",
    "description": "In metrics.csv (comma delimited), print line 1 (header), and for subsequent lines print only where $2 > 50.",
    "objective": "awk -F',' 'NR==1 || $2 > 50' metrics.csv",
    "hints": [
      "Command: awk -F',' 'NR==1 || $2 > 50' metrics.csv"
    ],
    "solution": "awk -F',' 'NR==1 || $2 > 50' metrics.csv",
    "accepted_regex": [
      "awk\\s+-F['\\\"]?,['\\\"]?\\s+['\\\"]NR\\s*==\\s*1\\s*\\|\\|\\s*\\$2\\s*>\\s*50['\\\"]\\s+metrics\\.csv"
    ],
    "setup_files": {
      "metrics.csv": "server,load\nsrv1,20\nsrv2,85\nsrv3,90\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-15",
    "category": "Awk Processing",
    "title": "Calculate Column Value Differences",
    "difficulty": "Intermediate",
    "description": "Print $1 and the difference ($3 - $2) from numbers.tsv.",
    "objective": "awk '{print $1, $3 - $2}' numbers.tsv",
    "hints": [
      "Command: awk '{print $1, $3 - $2}' numbers.tsv"
    ],
    "solution": "awk '{print $1, $3 - $2}' numbers.tsv",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{print\\s+\\$1,\\s*\\$3\\s*-\\s*\\$2\\}['\\\"]\\s+numbers\\.tsv"
    ],
    "setup_files": {
      "numbers.tsv": "row1 10 30\nrow2 20 50\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-16",
    "category": "Awk Processing",
    "title": "Find Maximum Value in Column 2",
    "difficulty": "Intermediate",
    "description": "Find and print the highest number in column 2 of stats.txt.",
    "objective": "awk '$2 > max {max = $2} END {print max}' stats.txt",
    "hints": [
      "Command: awk '$2 > max {max = $2} END {print max}' stats.txt"
    ],
    "solution": "awk '$2 > max {max = $2} END {print max}' stats.txt",
    "accepted_regex": [
      "awk\\s+['\\\"].*\\$2\\s*>\\s*max.*print\\s+max.*stats\\.txt"
    ],
    "setup_files": {
      "stats.txt": "a 15\nb 95\nc 40\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-17",
    "category": "Awk Processing",
    "title": "Substr Function to Extract Date Substring",
    "difficulty": "Advanced",
    "description": "Extract first 10 characters (YYYY-MM-DD) from timestamp column 1 using substr($1, 1, 10).",
    "objective": "awk '{print substr($1, 1, 10)}' logs.txt",
    "hints": [
      "Command: awk '{print substr($1, 1, 10)}' logs.txt"
    ],
    "solution": "awk '{print substr($1, 1, 10)}' logs.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{print\\s+substr\\(\\$1,\\s*1,\\s*10\\)\\}['\\\"]\\s+logs\\.txt"
    ],
    "setup_files": {
      "logs.txt": "2026-10-09T12:34:56Z event1\n2026-10-09T13:00:00Z event2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-18",
    "category": "Awk Processing",
    "title": "Count Occurrences Where Field Equals Target",
    "difficulty": "Easy",
    "description": "Count how many rows have column 2 equal to 'SUCCESS' in status.txt.",
    "objective": "awk '$2 == \"SUCCESS\" {count++} END {print count}' status.txt",
    "hints": [
      "Command: awk '$2 == \"SUCCESS\" {count++} END {print count}' status.txt"
    ],
    "solution": "awk '$2 == \"SUCCESS\" {count++} END {print count}' status.txt",
    "accepted_regex": [
      "awk\\s+['\\\"].*\\$2\\s*==\\s*[\\\"']SUCCESS[\\\"'].*count.*status\\.txt"
    ],
    "setup_files": {
      "status.txt": "job1 SUCCESS\njob2 FAILED\njob3 SUCCESS\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-19",
    "category": "Awk Processing",
    "title": "Print Range of Records from NR 5 to 8",
    "difficulty": "Easy",
    "description": "Print records where NR is between 5 and 8 (inclusive).",
    "objective": "awk 'NR >= 5 && NR <= 8' records.txt",
    "hints": [
      "Command: awk 'NR >= 5 && NR <= 8' records.txt"
    ],
    "solution": "awk 'NR >= 5 && NR <= 8' records.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]NR\\s*>=\\s*5\\s*&&\\s*NR\\s*<=\\s*8['\\\"]\\s+records\\.txt"
    ],
    "setup_files": {
      "records.txt": "line 1\nline 2\nline 3\nline 4\nline 5\nline 6\nline 7\nline 8\nline 9\nline 10\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-20",
    "category": "Awk Processing",
    "title": "Replace Field 2 Value Before Printing",
    "difficulty": "Intermediate",
    "description": "In data.txt, change $2 to 'REDACTED' and print the modified full line.",
    "objective": "awk '{$2 = \"REDACTED\"; print}' data.txt",
    "hints": [
      "Command: awk '{$2 = \"REDACTED\"; print}' data.txt"
    ],
    "solution": "awk '{$2 = \"REDACTED\"; print}' data.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{\\s*\\$2\\s*=\\s*[\\\"']REDACTED[\\\"'];?\\s*print\\}['\\\"]\\s+data\\.txt"
    ],
    "setup_files": {
      "data.txt": "user secret pass\nadmin secret pass\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-21",
    "category": "Awk Processing",
    "title": "Print Second to Last Field $(NF-1)",
    "difficulty": "Intermediate",
    "description": "Extract the second-to-last field $(NF-1) from variable length rows in items.txt.",
    "objective": "awk '{print $(NF-1)}' items.txt",
    "hints": [
      "Command: awk '{print $(NF-1)}' items.txt"
    ],
    "solution": "awk '{print $(NF-1)}' items.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{print\\s+\\$\\(NF-1\\)\\}['\\\"]\\s+items\\.txt"
    ],
    "setup_files": {
      "items.txt": "a b c d\nx y z\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-22",
    "category": "Awk Processing",
    "title": "Case-Insensitive Match in Awk using tolower",
    "difficulty": "Intermediate",
    "description": "Print rows in auth.log where tolower($0) matches 'error'.",
    "objective": "awk 'tolower($0) ~ /error/' auth.log",
    "hints": [
      "Command: awk 'tolower($0) ~ /error/' auth.log"
    ],
    "solution": "awk 'tolower($0) ~ /error/' auth.log",
    "accepted_regex": [
      "awk\\s+['\\\"]tolower\\(\\$0\\)\\s*~\\s*\\/error\\/['\\\"]\\s+auth\\.log"
    ],
    "setup_files": {
      "auth.log": "ERROR: fail\nInfo: ok\nError: minor\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-23",
    "category": "Awk Processing",
    "title": "Multiply Two Columns and Sum Total",
    "difficulty": "Intermediate",
    "description": "Calculate total revenue by summing ($2 * $3) (quantity * price) in orders.tsv.",
    "objective": "awk '{total += $2 * $3} END {print total}' orders.tsv",
    "hints": [
      "Command: awk '{total += $2 * $3} END {print total}' orders.tsv"
    ],
    "solution": "awk '{total += $2 * $3} END {print total}' orders.tsv",
    "accepted_regex": [
      "awk\\s+['\\\"].*total\\s*\\+=\\s*\\$2\\s*\\*\\s*\\$3.*print\\s+total.*orders\\.tsv"
    ],
    "setup_files": {
      "orders.tsv": "item1 2 10\nitem2 5 20\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-24",
    "category": "Awk Processing",
    "title": "Print Unique Entries in Column 1 Without External Sort",
    "difficulty": "Advanced",
    "description": "Print unique entries from column 1 of dups.txt preserving first appearance.",
    "objective": "awk '!seen[$1]++' dups.txt",
    "hints": [
      "Command: awk '!seen[$1]++' dups.txt"
    ],
    "solution": "awk '!seen[$1]++' dups.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]!seen\\[\\$1\\]\\+\\+['\\\"]\\s+dups\\.txt"
    ],
    "setup_files": {
      "dups.txt": "a 1\nb 2\na 3\nc 4\nb 5\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-25",
    "category": "Awk Processing",
    "title": "Sum Columns from df -h Output in Megabytes",
    "difficulty": "Intermediate",
    "description": "Sum column 4 (available blocks) in df.txt skipping the first header line.",
    "objective": "awk 'NR > 1 {sum += $4} END {print sum}' df.txt",
    "hints": [
      "Command: awk 'NR > 1 {sum += $4} END {print sum}' df.txt"
    ],
    "solution": "awk 'NR > 1 {sum += $4} END {print sum}' df.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]NR\\s*>\\s*1\\s*\\{\\s*sum\\s*\\+=\\s*\\$4.*print\\s+sum.*df\\.txt"
    ],
    "setup_files": {
      "df.txt": "Filesystem 1K-blocks Used Available Use% Mounted\n/dev/sda1 1000 200 800 20% /\n/dev/sda2 2000 500 1500 25% /home\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-01",
    "category": "Find & Permissions",
    "title": "Locate Insecure World-Writable Files",
    "difficulty": "Intermediate",
    "description": "Search './target_dir' for regular files with permissions 0777 (world-writable).",
    "objective": "find ./target_dir -type f -perm 0777",
    "hints": [
      "Command: find ./target_dir -type f -perm 0777"
    ],
    "solution": "find ./target_dir -type f -perm 0777",
    "accepted_regex": [
      "find\\s+(\\.\\/)?target_dir\\s+(-type\\s+f\\s+-perm\\s+(0?777)|-perm\\s+(0?777)\\s+-type\\s+f)"
    ],
    "setup_files": {
      "target_dir/safe.sh": "ok",
      "target_dir/insecure.tmp": "bad"
    },
    "setup_perms": {
      "target_dir/safe.sh": 493,
      "target_dir/insecure.tmp": 511
    },
    "verify_cmd": ""
  },
  {
    "id": "find-02",
    "category": "Find & Permissions",
    "title": "Find Log Files Modified Within Last 24 Hours",
    "difficulty": "Intermediate",
    "description": "Search './var_logs' for files ending in '.log' modified within the last 1 day.",
    "objective": "find ./var_logs -name '*.log' -mtime -1",
    "hints": [
      "Command: find ./var_logs -name '*.log' -mtime -1"
    ],
    "solution": "find ./var_logs -name '*.log' -mtime -1",
    "accepted_regex": [
      "find\\s+(\\.\\/)?var_logs\\s+.*-name\\s+['\\\"]?\\*\\.log['\\\"]?.*-mtime\\s+-1"
    ],
    "setup_files": {
      "var_logs/app.log": "today",
      "var_logs/old.txt": "text"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-03",
    "category": "Find & Permissions",
    "title": "Find SUID Binaries on System",
    "difficulty": "Advanced",
    "description": "Search './bin_dir' for regular files that have SUID (octal 4000) permission bit set.",
    "objective": "find ./bin_dir -type f -perm -4000",
    "hints": [
      "Command: find ./bin_dir -type f -perm -4000"
    ],
    "solution": "find ./bin_dir -type f -perm -4000",
    "accepted_regex": [
      "find\\s+(\\.\\/)?bin_dir\\s+(-type\\s+f\\s+-perm\\s+[-/]4000|-perm\\s+[-/]4000\\s+-type\\s+f)"
    ],
    "setup_files": {
      "bin_dir/suid_prog": "bin",
      "bin_dir/norm_prog": "bin"
    },
    "setup_perms": {
      "bin_dir/suid_prog": 2541,
      "bin_dir/norm_prog": 493
    },
    "verify_cmd": ""
  },
  {
    "id": "find-04",
    "category": "Find & Permissions",
    "title": "Find Files Larger Than 10 Megabytes",
    "difficulty": "Intermediate",
    "description": "Search './data_store' for regular files strictly larger than 10 Megabytes (+10M).",
    "objective": "find ./data_store -type f -size +10M",
    "hints": [
      "Command: find ./data_store -type f -size +10M"
    ],
    "solution": "find ./data_store -type f -size +10M",
    "accepted_regex": [
      "find\\s+(\\.\\/)?data_store\\s+(-type\\s+f\\s+-size\\s+\\+10M|-size\\s+\\+10M\\s+-type\\s+f)"
    ],
    "setup_files": {
      "data_store/s.txt": "small",
      "data_store/b.img": "__SPARSE_BYTES:12582912"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-05",
    "category": "Find & Permissions",
    "title": "Find All Empty Directories",
    "difficulty": "Easy",
    "description": "Search './tree' for directories that contain no files or subdirectories.",
    "objective": "find ./tree -type d -empty",
    "hints": [
      "Command: find ./tree -type d -empty"
    ],
    "solution": "find ./tree -type d -empty",
    "accepted_regex": [
      "find\\s+(\\.\\/)?tree\\s+(-type\\s+d\\s+-empty|-empty\\s+-type\\s+d)"
    ],
    "setup_files": {
      "tree/not_empty/a": "1",
      "tree/empty_dir/.keep": ""
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-06",
    "category": "Find & Permissions",
    "title": "Find and Execute Chmod in Batches",
    "difficulty": "Intermediate",
    "description": "Find all *.sh files under './scripts' and chmod them to 755 using -exec ... {} +.",
    "objective": "find ./scripts -type f -name '*.sh' -exec chmod 755 {} +",
    "hints": [
      "Command: find ./scripts -type f -name '*.sh' -exec chmod 755 {} +"
    ],
    "solution": "find ./scripts -type f -name '*.sh' -exec chmod 755 {} +",
    "accepted_regex": [
      "find\\s+(\\.\\/)?scripts\\s+.*-name\\s+['\\\"]?\\*\\.sh['\\\"]?\\s+-exec\\s+chmod\\s+755\\s+\\{\\}\\s+\\+"
    ],
    "setup_files": {
      "scripts/run.sh": "echo 1\n",
      "scripts/test.sh": "echo 2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-07",
    "category": "Find & Permissions",
    "title": "Find Files Owned by Specific User",
    "difficulty": "Easy",
    "description": "Search directory './home' for files owned by user 'nobody'.",
    "objective": "find ./home -user nobody",
    "hints": [
      "Command: find ./home -user nobody"
    ],
    "solution": "find ./home -user nobody",
    "accepted_regex": [
      "find\\s+(\\.\\/)?home\\s+.*-user\\s+nobody"
    ],
    "setup_files": {
      "home/user_file": "data"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-08",
    "category": "Find & Permissions",
    "title": "Find SGID Binaries on Directory",
    "difficulty": "Advanced",
    "description": "Search './bin_dir' for files with SGID bit set (octal 2000).",
    "objective": "find ./bin_dir -type f -perm -2000",
    "hints": [
      "Command: find ./bin_dir -type f -perm -2000"
    ],
    "solution": "find ./bin_dir -type f -perm -2000",
    "accepted_regex": [
      "find\\s+(\\.\\/)?bin_dir\\s+.*-perm\\s+[-/]2000"
    ],
    "setup_files": {
      "bin_dir/sgid_bin": "b"
    },
    "setup_perms": {
      "bin_dir/sgid_bin": 1517
    },
    "verify_cmd": ""
  },
  {
    "id": "find-09",
    "category": "Find & Permissions",
    "title": "Find Files Modified in Last 30 Minutes",
    "difficulty": "Intermediate",
    "description": "Locate regular files in /tmp modified less than 30 minutes ago (-mmin -30).",
    "objective": "find /tmp -type f -mmin -30",
    "hints": [
      "Command: find /tmp -type f -mmin -30"
    ],
    "solution": "find /tmp -type f -mmin -30",
    "accepted_regex": [
      "find\\s+\\/tmp\\s+(-type\\s+f\\s+-mmin\\s+-30|-mmin\\s+-30\\s+-type\\s+f)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-10",
    "category": "Find & Permissions",
    "title": "Limit Search Depth to Maximum 2 Directories",
    "difficulty": "Easy",
    "description": "Find all .conf files under './etc' without descending more than 2 levels deep.",
    "objective": "find ./etc -maxdepth 2 -name '*.conf'",
    "hints": [
      "Command: find ./etc -maxdepth 2 -name '*.conf'"
    ],
    "solution": "find ./etc -maxdepth 2 -name '*.conf'",
    "accepted_regex": [
      "find\\s+(\\.\\/)?etc\\s+.*-maxdepth\\s+2.*-name\\s+['\\\"]?\\*\\.conf['\\\"]?"
    ],
    "setup_files": {
      "etc/app.conf": "1",
      "etc/sub/deep.conf": "2"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-11",
    "category": "Find & Permissions",
    "title": "Find Broken Symbolic Links",
    "difficulty": "Advanced",
    "description": "Find broken/dangling symbolic links in directory './links'.",
    "objective": "find ./links -xtype l",
    "hints": [
      "Command: find ./links -xtype l"
    ],
    "solution": "find ./links -xtype l",
    "accepted_regex": [
      "find\\s+(\\.\\/)?links\\s+.*-xtype\\s+l"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-12",
    "category": "Find & Permissions",
    "title": "Case-Insensitive Filename Search",
    "difficulty": "Easy",
    "description": "Search directory './docs' for files matching 'readme.txt' case-insensitively.",
    "objective": "find ./docs -iname 'readme.txt'",
    "hints": [
      "Command: find ./docs -iname 'readme.txt'"
    ],
    "solution": "find ./docs -iname 'readme.txt'",
    "accepted_regex": [
      "find\\s+(\\.\\/)?docs\\s+.*-iname\\s+['\\\"]?readme\\.txt['\\\"]?"
    ],
    "setup_files": {
      "docs/README.TXT": "docs"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-13",
    "category": "Find & Permissions",
    "title": "Find Files with Read-Only Permissions 0444",
    "difficulty": "Intermediate",
    "description": "Search './archive' for files with exact permissions 0444 (read-only for all).",
    "objective": "find ./archive -type f -perm 0444",
    "hints": [
      "Command: find ./archive -type f -perm 0444"
    ],
    "solution": "find ./archive -type f -perm 0444",
    "accepted_regex": [
      "find\\s+(\\.\\/)?archive\\s+.*-perm\\s+(0?444)"
    ],
    "setup_files": {
      "archive/locked.txt": "1"
    },
    "setup_perms": {
      "archive/locked.txt": 292
    },
    "verify_cmd": ""
  },
  {
    "id": "find-14",
    "category": "Find & Permissions",
    "title": "Find and Delete Files Matching Pattern Directly",
    "difficulty": "Intermediate",
    "description": "Find all *.tmp files in './scratch' and delete them using find's -delete action.",
    "objective": "find ./scratch -type f -name '*.tmp' -delete",
    "hints": [
      "Command: find ./scratch -type f -name '*.tmp' -delete"
    ],
    "solution": "find ./scratch -type f -name '*.tmp' -delete",
    "accepted_regex": [
      "find\\s+(\\.\\/)?scratch\\s+.*-name\\s+['\\\"]?\\*\\.tmp['\\\"]?\\s+-delete"
    ],
    "setup_files": {
      "scratch/temp1.tmp": "1"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-15",
    "category": "Find & Permissions",
    "title": "Find Files Accessed More Than 30 Days Ago",
    "difficulty": "Intermediate",
    "description": "Locate files in './cache' that have not been accessed in over 30 days (-atime +30).",
    "objective": "find ./cache -type f -atime +30",
    "hints": [
      "Command: find ./cache -type f -atime +30"
    ],
    "solution": "find ./cache -type f -atime +30",
    "accepted_regex": [
      "find\\s+(\\.\\/)?cache\\s+.*-atime\\s+\\+30"
    ],
    "setup_files": {
      "cache/old.bin": "bin"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-16",
    "category": "Find & Permissions",
    "title": "Find Directories with Sticky Bit Set",
    "difficulty": "Advanced",
    "description": "Search /tmp for directories having the sticky bit set (octal 1000).",
    "objective": "find /tmp -type d -perm -1000",
    "hints": [
      "Command: find /tmp -type d -perm -1000"
    ],
    "solution": "find /tmp -type d -perm -1000",
    "accepted_regex": [
      "find\\s+\\/tmp\\s+(-type\\s+d\\s+-perm\\s+[-/]1000|-perm\\s+[-/]1000\\s+-type\\s+d)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-17",
    "category": "Find & Permissions",
    "title": "Find Regular Files Between 1M and 5M in Size",
    "difficulty": "Intermediate",
    "description": "Locate files under './media' whose size is between 1 Megabyte and 5 Megabytes.",
    "objective": "find ./media -type f -size +1M -size -5M",
    "hints": [
      "Command: find ./media -type f -size +1M -size -5M"
    ],
    "solution": "find ./media -type f -size +1M -size -5M",
    "accepted_regex": [
      "find\\s+(\\.\\/)?media\\s+.*-size\\s+\\+1M\\s+-size\\s+-5M"
    ],
    "setup_files": {
      "media/song.mp3": "__SPARSE_BYTES:2621440"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-18",
    "category": "Find & Permissions",
    "title": "Find Files Not Owned by Current User",
    "difficulty": "Intermediate",
    "description": "Search './shared' for files not owned by root (! -user root).",
    "objective": "find ./shared ! -user root",
    "hints": [
      "Command: find ./shared ! -user root"
    ],
    "solution": "find ./shared ! -user root",
    "accepted_regex": [
      "find\\s+(\\.\\/)?shared\\s+(!\\s+-user\\s+root|-not\\s+-user\\s+root)"
    ],
    "setup_files": {
      "shared/data": "content"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-19",
    "category": "Find & Permissions",
    "title": "Find and Print Formatted Output with -printf",
    "difficulty": "Advanced",
    "description": "Find regular files in './logs' and print '%p: %s bytes\\n'.",
    "objective": "find ./logs -type f -printf \"%p: %s bytes\\n\"",
    "hints": [
      "Command: find ./logs -type f -printf \"%p: %s bytes\\n\""
    ],
    "solution": "find ./logs -type f -printf \"%p: %s bytes\\n\"",
    "accepted_regex": [
      "find\\s+(\\.\\/)?logs\\s+.*-printf\\s+[\"\\'].*%p.*%s"
    ],
    "setup_files": {
      "logs/sys.log": "data"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-20",
    "category": "Find & Permissions",
    "title": "Find Files Matching Either of Two Names",
    "difficulty": "Intermediate",
    "description": "Search './project' for files named either '*.c' or '*.h' using -o (OR).",
    "objective": "find ./project -name '*.c' -o -name '*.h'",
    "hints": [
      "Command: find ./project -name '*.c' -o -name '*.h'"
    ],
    "solution": "find ./project -name '*.c' -o -name '*.h'",
    "accepted_regex": [
      "find\\s+(\\.\\/)?project\\s+.*-name\\s+['\\\"]?\\*\\.c['\\\"]\\s+-o\\s+-name\\s+['\\\"]?\\*\\.h['\\\"]"
    ],
    "setup_files": {
      "project/main.c": "1",
      "project/main.h": "2"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-01",
    "category": "Pipes & Redirections",
    "title": "Redirect stdout and stderr into audit.log",
    "difficulty": "Intermediate",
    "description": "Execute './deploy_check.sh' redirecting BOTH stdout (fd 1) and stderr (fd 2) into audit.log.",
    "objective": "./deploy_check.sh > audit.log 2>&1",
    "hints": [
      "Command: ./deploy_check.sh > audit.log 2>&1"
    ],
    "solution": "./deploy_check.sh > audit.log 2>&1",
    "accepted_regex": [
      "\\.\\/deploy_check\\.sh\\s+>\\s*audit\\.log\\s+2>&1",
      "\\.\\/deploy_check\\.sh\\s+&>\\s*audit\\.log"
    ],
    "setup_files": {
      "deploy_check.sh": "#!/bin/bash\necho out\necho err >&2\n"
    },
    "setup_perms": {
      "deploy_check.sh": 493
    },
    "verify_cmd": ""
  },
  {
    "id": "pipe-02",
    "category": "Pipes & Redirections",
    "title": "Count Frequency and Sort Top 3",
    "difficulty": "Intermediate",
    "description": "Extract column 1 (IP) from connections.log, count unique IPs, and sort top 3 descending.",
    "objective": "awk '{print $1}' connections.log | sort | uniq -c | sort -nr | head -n 3",
    "hints": [
      "Command: awk '{print $1}' connections.log | sort | uniq -c | sort -nr | head -n 3"
    ],
    "solution": "awk '{print $1}' connections.log | sort | uniq -c | sort -nr | head -n 3",
    "accepted_regex": [
      "(awk\\s+['\\\"]\\{print\\s+\\$1\\}['\\\"]|cut\\s+-d['\\\"]\\s*['\\\"]\\s+-f1)\\s+connections\\.log\\s*\\|\\s*sort\\s*\\|\\s*uniq\\s+-c\\s*\\|\\s*sort\\s+(-nr|-rn|-n\\s+-r|-r\\s+-n)\\s*\\|\\s*head\\s+(-n\\s*3|-3)"
    ],
    "setup_files": {
      "connections.log": "10.0.0.1 a\n10.0.0.2 b\n10.0.0.1 c\n10.0.0.1 d\n10.0.0.2 e\n10.0.0.3 f\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-03",
    "category": "Pipes & Redirections",
    "title": "Tee Output to File While Filtering in Pipeline",
    "difficulty": "Intermediate",
    "description": "Cat events.log, tee unfiltered output to backup_events.log, and filter stream with grep ERROR.",
    "objective": "cat events.log | tee backup_events.log | grep ERROR",
    "hints": [
      "Command: cat events.log | tee backup_events.log | grep ERROR"
    ],
    "solution": "cat events.log | tee backup_events.log | grep ERROR",
    "accepted_regex": [
      "cat\\s+events\\.log\\s*\\|\\s*tee\\s+backup_events\\.log\\s*\\|\\s*grep\\s+ERROR"
    ],
    "setup_files": {
      "events.log": "INFO 1\nERROR 2\nINFO 3\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-04",
    "category": "Pipes & Redirections",
    "title": "Sort Numerical Column in Reverse with Delimiter",
    "difficulty": "Intermediate",
    "description": "Read disk_usage.txt. Sort by column 2 numerically in reverse order (-k2 -nr).",
    "objective": "sort -k2 -nr disk_usage.txt",
    "hints": [
      "Command: sort -k2 -nr disk_usage.txt"
    ],
    "solution": "sort -k2 -nr disk_usage.txt",
    "accepted_regex": [
      "sort\\s+.*-k\\s*2.*-nr\\s+disk_usage\\.txt",
      "sort\\s+.*-k\\s*2\\s+-n\\s+-r\\s+disk_usage\\.txt"
    ],
    "setup_files": {
      "disk_usage.txt": "/var 500\n/home 1000\n/tmp 100\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-05",
    "category": "Pipes & Redirections",
    "title": "Skip First Header Line Before Sorting",
    "difficulty": "Intermediate",
    "description": "Read metrics.csv. Skip header with 'tail -n +2' and sort remaining by column 2 numerically.",
    "objective": "tail -n +2 metrics.csv | sort -t',' -k2 -n",
    "hints": [
      "Command: tail -n +2 metrics.csv | sort -t',' -k2 -n"
    ],
    "solution": "tail -n +2 metrics.csv | sort -t',' -k2 -n",
    "accepted_regex": [
      "tail\\s+-n\\s+\\+2\\s+metrics\\.csv\\s*\\|\\s*sort\\s+.*-k\\s*2"
    ],
    "setup_files": {
      "metrics.csv": "server,load\nsrv1,90\nsrv2,20\nsrv3,50\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-06",
    "category": "Pipes & Redirections",
    "title": "Discard Error Output to /dev/null",
    "difficulty": "Easy",
    "description": "Run 'find / -name secret.txt', redirecting all standard error to /dev/null.",
    "objective": "find / -name secret.txt 2>/dev/null",
    "hints": [
      "Command: find / -name secret.txt 2>/dev/null"
    ],
    "solution": "find / -name secret.txt 2>/dev/null",
    "accepted_regex": [
      "find\\s+\\/\\s+-name\\s+secret\\.txt\\s+2>\\/dev\\/null"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-07",
    "category": "Pipes & Redirections",
    "title": "Append Standard Output Without Overwriting",
    "difficulty": "Easy",
    "description": "Append date timestamp to deploy.log without truncating existing content.",
    "objective": "date >> deploy.log",
    "hints": [
      "Use >> to append",
      "Command: date >> deploy.log"
    ],
    "solution": "date >> deploy.log",
    "accepted_regex": [
      "date\\s+>>\\s*deploy\\.log"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-08",
    "category": "Pipes & Redirections",
    "title": "Pass Word List to Xargs Command",
    "difficulty": "Intermediate",
    "description": "Cat packages.txt and pass package names to 'rpm -q' using xargs.",
    "objective": "cat packages.txt | xargs rpm -q",
    "hints": [
      "Command: cat packages.txt | xargs rpm -q"
    ],
    "solution": "cat packages.txt | xargs rpm -q",
    "accepted_regex": [
      "cat\\s+packages\\.txt\\s*\\|\\s*xargs\\s+rpm\\s+-q"
    ],
    "setup_files": {
      "packages.txt": "bash\ncoreutils\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-09",
    "category": "Pipes & Redirections",
    "title": "Cut Specific Delimited Fields",
    "difficulty": "Easy",
    "description": "Extract columns 1 and 3 from comma-delimited users.csv using cut.",
    "objective": "cut -d',' -f1,3 users.csv",
    "hints": [
      "Command: cut -d',' -f1,3 users.csv"
    ],
    "solution": "cut -d',' -f1,3 users.csv",
    "accepted_regex": [
      "cut\\s+-d['\\\"]?,['\\\"]?\\s+-f1,3\\s+users\\.csv"
    ],
    "setup_files": {
      "users.csv": "alice,pass1,admin\nbob,pass2,user\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-10",
    "category": "Pipes & Redirections",
    "title": "Translate Characters to Uppercase with tr",
    "difficulty": "Easy",
    "description": "Cat input.txt and convert all lowercase letters to uppercase using tr '[:lower:]' '[:upper:]'.",
    "objective": "cat input.txt | tr '[:lower:]' '[:upper:]'",
    "hints": [
      "Command: cat input.txt | tr '[:lower:]' '[:upper:]'"
    ],
    "solution": "cat input.txt | tr '[:lower:]' '[:upper:]'",
    "accepted_regex": [
      "cat\\s+input\\.txt\\s*\\|\\s*tr\\s+.*'\\[:lower:\\]'.*'\\[:upper:\\]'"
    ],
    "setup_files": {
      "input.txt": "hello world\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-11",
    "category": "Pipes & Redirections",
    "title": "Count Total Lines in Pipeline with wc -l",
    "difficulty": "Easy",
    "description": "Count how many processes are listed by 'ps -ef' (piped into wc -l).",
    "objective": "ps -ef | wc -l",
    "hints": [
      "Command: ps -ef | wc -l"
    ],
    "solution": "ps -ef | wc -l",
    "accepted_regex": [
      "ps\\s+(-ef|-aux|aux)\\s*\\|\\s*wc\\s+-l"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-12",
    "category": "Pipes & Redirections",
    "title": "Display Only Duplicate Lines with uniq -d",
    "difficulty": "Intermediate",
    "description": "Read sorted list items.txt and output only the lines that appear more than once (uniq -d).",
    "objective": "uniq -d items.txt",
    "hints": [
      "Command: uniq -d items.txt"
    ],
    "solution": "uniq -d items.txt",
    "accepted_regex": [
      "uniq\\s+-d\\s+items\\.txt"
    ],
    "setup_files": {
      "items.txt": "a\nb\nb\nc\nd\nd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-13",
    "category": "Pipes & Redirections",
    "title": "Display Only Unique Non-Repeated Lines with uniq -u",
    "difficulty": "Intermediate",
    "description": "Read sorted list items.txt and output only lines that appear exactly once (uniq -u).",
    "objective": "uniq -u items.txt",
    "hints": [
      "Command: uniq -u items.txt"
    ],
    "solution": "uniq -u items.txt",
    "accepted_regex": [
      "uniq\\s+-u\\s+items\\.txt"
    ],
    "setup_files": {
      "items.txt": "a\nb\nb\nc\nd\nd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-14",
    "category": "Pipes & Redirections",
    "title": "Combine stdout of Command into Variable with Command Substitution",
    "difficulty": "Easy",
    "description": "Assign the current date string into variable CURRENT_DATE.",
    "objective": "CURRENT_DATE=$(date)",
    "hints": [
      "Command: CURRENT_DATE=$(date)"
    ],
    "solution": "CURRENT_DATE=$(date)",
    "accepted_regex": [
      "CURRENT_DATE=\\$\\(date\\)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-15",
    "category": "Pipes & Redirections",
    "title": "Append Output with Tee -a",
    "difficulty": "Intermediate",
    "description": "Echo 'NEW_ENTRY' and append it to status.log while viewing it in console with tee -a.",
    "objective": "echo 'NEW_ENTRY' | tee -a status.log",
    "hints": [
      "Command: echo 'NEW_ENTRY' | tee -a status.log"
    ],
    "solution": "echo 'NEW_ENTRY' | tee -a status.log",
    "accepted_regex": [
      "echo\\s+['\\\"]?NEW_ENTRY['\\\"]?\\s*\\|\\s*tee\\s+-a\\s+status\\.log"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-16",
    "category": "Pipes & Redirections",
    "title": "Paste Two Files Side by Side with Tab Separator",
    "difficulty": "Easy",
    "description": "Merge lines of col1.txt and col2.txt side-by-side using paste.",
    "objective": "paste col1.txt col2.txt",
    "hints": [
      "Command: paste col1.txt col2.txt"
    ],
    "solution": "paste col1.txt col2.txt",
    "accepted_regex": [
      "paste\\s+col1\\.txt\\s+col2\\.txt"
    ],
    "setup_files": {
      "col1.txt": "a\nb\n",
      "col2.txt": "1\n2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-17",
    "category": "Pipes & Redirections",
    "title": "Filter Unique Lines Ignoring Case with uniq -i",
    "difficulty": "Intermediate",
    "description": "Filter adjacent duplicate lines from case_items.txt ignoring case differences.",
    "objective": "uniq -i case_items.txt",
    "hints": [
      "Command: uniq -i case_items.txt"
    ],
    "solution": "uniq -i case_items.txt",
    "accepted_regex": [
      "uniq\\s+-i\\s+case_items\\.txt"
    ],
    "setup_files": {
      "case_items.txt": "apple\nApple\nbanana\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-18",
    "category": "Pipes & Redirections",
    "title": "Sort by IP Address Numerically with sort -V or sort -n",
    "difficulty": "Intermediate",
    "description": "Sort ip_list.txt by natural version/numeric order.",
    "objective": "sort -V ip_list.txt",
    "hints": [
      "Command: sort -V ip_list.txt"
    ],
    "solution": "sort -V ip_list.txt",
    "accepted_regex": [
      "sort\\s+-V\\s+ip_list\\.txt"
    ],
    "setup_files": {
      "ip_list.txt": "10.0.0.10\n10.0.0.2\n10.0.0.1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-19",
    "category": "Pipes & Redirections",
    "title": "Delete Specific Characters with tr -d",
    "difficulty": "Easy",
    "description": "Strip all comma (,) characters from text.txt using tr -d.",
    "objective": "cat text.txt | tr -d ','",
    "hints": [
      "Command: cat text.txt | tr -d ','"
    ],
    "solution": "cat text.txt | tr -d ','",
    "accepted_regex": [
      "cat\\s+text\\.txt\\s*\\|\\s*tr\\s+-d\\s+['\\\"],['\\\"]"
    ],
    "setup_files": {
      "text.txt": "1,000,000\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-20",
    "category": "Pipes & Redirections",
    "title": "Execute Parallel Tasks with xargs -P",
    "difficulty": "Advanced",
    "description": "Cat urls.txt and fetch headers in parallel using 4 workers with xargs -n 1 -P 4.",
    "objective": "cat urls.txt | xargs -n 1 -P 4 curl -sI",
    "hints": [
      "Command: cat urls.txt | xargs -n 1 -P 4 curl -sI"
    ],
    "solution": "cat urls.txt | xargs -n 1 -P 4 curl -sI",
    "accepted_regex": [
      "cat\\s+urls\\.txt\\s*\\|\\s*xargs\\s+.*-P\\s*4\\s+curl"
    ],
    "setup_files": {
      "urls.txt": "http://localhost\nhttp://127.0.0.1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-21",
    "category": "Pipes & Redirections",
    "title": "Sort Colon-Delimited File by Numeric Column 3",
    "difficulty": "Intermediate",
    "description": "Sort /etc/passwd by UID (column 3) in numeric ascending order.",
    "objective": "sort -t':' -k3 -n /etc/passwd",
    "hints": [
      "Command: sort -t':' -k3 -n /etc/passwd"
    ],
    "solution": "sort -t':' -k3 -n /etc/passwd",
    "accepted_regex": [
      "sort\\s+-t['\\\"]?:['\\\"]?\\s+-k3\\s+-n\\s+\\/etc\\/passwd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-22",
    "category": "Pipes & Redirections",
    "title": "Find Common Lines Between Two Sorted Files with comm",
    "difficulty": "Intermediate",
    "description": "Compare sorted fileA.txt and fileB.txt and output only the lines common to both (comm -12).",
    "objective": "comm -12 fileA.txt fileB.txt",
    "hints": [
      "Command: comm -12 fileA.txt fileB.txt"
    ],
    "solution": "comm -12 fileA.txt fileB.txt",
    "accepted_regex": [
      "comm\\s+-12\\s+fileA\\.txt\\s+fileB\\.txt"
    ],
    "setup_files": {
      "fileA.txt": "a\nb\nc\n",
      "fileB.txt": "b\nc\nd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-23",
    "category": "Pipes & Redirections",
    "title": "View Last 15 Lines of File with tail -n 15",
    "difficulty": "Easy",
    "description": "Display the last 15 lines of /var/log/messages.",
    "objective": "tail -n 15 /var/log/messages",
    "hints": [
      "Command: tail -n 15 /var/log/messages"
    ],
    "solution": "tail -n 15 /var/log/messages",
    "accepted_regex": [
      "tail\\s+(-n\\s*15|-15)\\s+\\/var\\/log\\/messages"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-24",
    "category": "Pipes & Redirections",
    "title": "View First 8 Lines of File with head -n 8",
    "difficulty": "Easy",
    "description": "Display the first 8 lines of /etc/hosts.",
    "objective": "head -n 8 /etc/hosts",
    "hints": [
      "Command: head -n 8 /etc/hosts"
    ],
    "solution": "head -n 8 /etc/hosts",
    "accepted_regex": [
      "head\\s+(-n\\s*8|-8)\\s+\\/etc\\/hosts"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-25",
    "category": "Pipes & Redirections",
    "title": "Redirect Standard Input with Here-Doc",
    "difficulty": "Intermediate",
    "description": "Use a bash here-doc (<<EOF) to pass multiple lines into cat and save to doc.txt.",
    "objective": "cat << 'EOF' > doc.txt\nline1\nline2\nEOF",
    "hints": [
      "Use cat << 'EOF' > doc.txt"
    ],
    "solution": "cat << 'EOF' > doc.txt\nline1\nline2\nEOF",
    "accepted_regex": [
      "cat\\s+<<.*doc\\.txt"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-21",
    "category": "Find & Permissions",
    "title": "Find Large Files Over 100MB",
    "difficulty": "Intermediate",
    "description": "Find all regular files in /var/log exceeding 100 Megabytes in size.",
    "objective": "find /var/log -type f -size +100M",
    "hints": [
      "Use -size +100M and -type f",
      "Command: find /var/log -type f -size +100M"
    ],
    "solution": "find /var/log -type f -size +100M",
    "accepted_regex": [
      "find\\s+\\/var\\/log\\s+.*-type\\s+f.*-size\\s+\\+100M",
      "find\\s+\\/var\\/log\\s+.*-size\\s+\\+100M.*-type\\s+f"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-22",
    "category": "Find & Permissions",
    "title": "Set Sticky Bit on Shared Temporary Directory",
    "difficulty": "Intermediate",
    "description": "Set the sticky bit on directory /shared/tmp (chmod +t or chmod 1777) so only file owners can delete their files.",
    "objective": "chmod +t /shared/tmp",
    "hints": [
      "Use chmod +t or chmod 1777",
      "Command: chmod +t /shared/tmp"
    ],
    "solution": "chmod +t /shared/tmp",
    "accepted_regex": [
      "chmod\\s+(\\+t|1[0-7]{3})\\s+\\/shared\\/tmp"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-23",
    "category": "Find & Permissions",
    "title": "Set SGID on Directory for Group Collaboration",
    "difficulty": "Intermediate",
    "description": "Set the SetGID (SGID) bit on /opt/project so all newly created files inherit the directory's owning group.",
    "objective": "chmod g+s /opt/project",
    "hints": [
      "Use chmod g+s or chmod 2775",
      "Command: chmod g+s /opt/project"
    ],
    "solution": "chmod g+s /opt/project",
    "accepted_regex": [
      "chmod\\s+(g\\+s|2[0-7]{3})\\s+\\/opt\\/project"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-24",
    "category": "Find & Permissions",
    "title": "Locate Broken or Dead Symlinks",
    "difficulty": "Intermediate",
    "description": "Find all broken symbolic links in /etc using find -xtype l.",
    "objective": "find /etc -xtype l",
    "hints": [
      "Use -xtype l to test target existence",
      "Command: find /etc -xtype l"
    ],
    "solution": "find /etc -xtype l",
    "accepted_regex": [
      "find\\s+\\/etc\\s+-xtype\\s+l"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-25",
    "category": "Find & Permissions",
    "title": "Find Files Modified in the Last 24 Hours",
    "difficulty": "Easy",
    "description": "Find all files in /etc modified within the last 24 hours (-mtime -1).",
    "objective": "find /etc -mtime -1",
    "hints": [
      "Use -mtime -1",
      "Command: find /etc -mtime -1"
    ],
    "solution": "find /etc -mtime -1",
    "accepted_regex": [
      "find\\s+\\/etc\\s+.*-mtime\\s+-1"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-31",
    "category": "Grep & Regex",
    "title": "Match Lines Beginning with One or More Digits",
    "difficulty": "Easy",
    "description": "Search 'data.txt' for all lines starting with one or more numerical digits.",
    "objective": "grep -E '^[0-9]+' data.txt",
    "hints": [
      "Use anchor ^ and [0-9]+ with -E",
      "Command: grep -E '^[0-9]+' data.txt"
    ],
    "solution": "grep -E '^[0-9]+' data.txt",
    "accepted_regex": [
      "grep\\s+(-E|--extended-regexp)\\s+['\\\"]?\\^[0-9]\\+['\\\"]?\\s+data\\.txt",
      "egrep\\s+['\\\"]?\\^[0-9]\\+['\\\"]?\\s+data\\.txt"
    ],
    "setup_files": {
      "data.txt": "100 apples\nbanana\n200 oranges\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-32",
    "category": "Grep & Regex",
    "title": "Recursive Grep Ignoring Binary Files",
    "difficulty": "Intermediate",
    "description": "Recursively search /etc for 'DB_PASSWORD' while ignoring binary files using -r -I.",
    "objective": "grep -rI 'DB_PASSWORD' /etc",
    "hints": [
      "Use -r (recursive) and -I (ignore binaries)",
      "Command: grep -rI 'DB_PASSWORD' /etc"
    ],
    "solution": "grep -rI 'DB_PASSWORD' /etc",
    "accepted_regex": [
      "grep\\s+(-rI|-Ir|-r\\s+-I|-I\\s+-r)\\s+['\\\"]?DB_PASSWORD['\\\"]?\\s+\\/etc"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-33",
    "category": "Grep & Regex",
    "title": "Extract All IPv4 Addresses from Log",
    "difficulty": "Intermediate",
    "description": "Extract only valid IPv4 address strings from network.log using grep -oE.",
    "objective": "grep -oE '([0-9]{1,3}\\.){3}[0-9]{1,3}' network.log",
    "hints": [
      "Use -o for only-matching and -E for regex",
      "Command: grep -oE '([0-9]{1,3}\\.){3}[0-9]{1,3}' network.log"
    ],
    "solution": "grep -oE '([0-9]{1,3}\\.){3}[0-9]{1,3}' network.log",
    "accepted_regex": [
      "grep\\s+(-oE|-Eo)\\s+['\\\"].*([0-9].*){3}.*['\\\"]\\s+network\\.log"
    ],
    "setup_files": {
      "network.log": "Connection from 192.168.1.50 port 22\nRefused 10.0.0.1 port 80\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-34",
    "category": "Grep & Regex",
    "title": "Count Exact Number of Matches Across Log",
    "difficulty": "Easy",
    "description": "Count the total number of lines in error.log containing 'FAIL' using grep -c.",
    "objective": "grep -c 'FAIL' error.log",
    "hints": [
      "Use grep -c 'FAIL' error.log",
      "Command: grep -c 'FAIL' error.log"
    ],
    "solution": "grep -c 'FAIL' error.log",
    "accepted_regex": [
      "grep\\s+-c\\s+['\\\"]?FAIL['\\\"]?\\s+error\\.log"
    ],
    "setup_files": {
      "error.log": "FAIL: disk\nOK\nFAIL: memory\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-35",
    "category": "Grep & Regex",
    "title": "Invert Search to Strip Comments and Blank Lines",
    "difficulty": "Intermediate",
    "description": "Display /etc/chrony.conf excluding all comments (starting with #) and empty lines.",
    "objective": "grep -vE '^#|^$' /etc/chrony.conf",
    "hints": [
      "Use -vE with pattern '^#|^$'",
      "Command: grep -vE '^#|^$' /etc/chrony.conf"
    ],
    "solution": "grep -vE '^#|^$' /etc/chrony.conf",
    "accepted_regex": [
      "grep\\s+(-vE|-Ev)\\s+['\\\"]?\\^#\\|\\^\\$['\\\"]?\\s+\\/etc\\/chrony\\.conf"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-26",
    "category": "Sed Stream Editing",
    "title": "Replace Entire Line Matching Pattern with 'c\\'",
    "difficulty": "Intermediate",
    "description": "In /etc/selinux/config, replace the entire line containing 'SELINUX=' with 'SELINUX=enforcing' using sed change command.",
    "objective": "sed -i '/^SELINUX=/c\\SELINUX=enforcing' /etc/selinux/config",
    "hints": [
      "Use /pattern/c\\replacement",
      "Command: sed -i '/^SELINUX=/c\\SELINUX=enforcing' /etc/selinux/config"
    ],
    "solution": "sed -i '/^SELINUX=/c\\SELINUX=enforcing' /etc/selinux/config",
    "accepted_regex": [
      "sed\\s+.*-i.*\\/(\\^)?SELINUX=\\/c.*SELINUX=enforcing.*\\/etc\\/selinux\\/config"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-27",
    "category": "Sed Stream Editing",
    "title": "Print Content Between START and END Markers",
    "difficulty": "Intermediate",
    "description": "Print only the lines between /START/ and /END/ inclusive from build.log using sed -n.",
    "objective": "sed -n '/START/,/END/p' build.log",
    "hints": [
      "Use range /START/,/END/p with -n",
      "Command: sed -n '/START/,/END/p' build.log"
    ],
    "solution": "sed -n '/START/,/END/p' build.log",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]\\/START\\/,\\/END\\/p['\\\"]\\s+build\\.log"
    ],
    "setup_files": {
      "build.log": "header\nSTART\nstep1\nstep2\nEND\nfooter\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-28",
    "category": "Sed Stream Editing",
    "title": "Delete Blank and Whitespace-Only Lines",
    "difficulty": "Easy",
    "description": "Remove all blank lines and lines containing only spaces or tabs from config.txt.",
    "objective": "sed '/^[[:space:]]*$/d' config.txt",
    "hints": [
      "Use pattern /^[[:space:]]*$/d",
      "Command: sed '/^[[:space:]]*$/d' config.txt"
    ],
    "solution": "sed '/^[[:space:]]*$/d' config.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/(\\^\\[\\[:space:\\]\\]\\*\\$|\\^\\s\\*\\$)\\/d['\\\"]\\s+config\\.txt"
    ],
    "setup_files": {
      "config.txt": "item1\n   \nitem2\n\nitem3\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-29",
    "category": "Sed Stream Editing",
    "title": "Case-Insensitive Global Substitution",
    "difficulty": "Easy",
    "description": "Replace all occurrences of 'localhost' (any case: Localhost, LOCALHOST) with '127.0.0.1' using sed flag 'I'.",
    "objective": "sed 's/localhost/127.0.0.1/gI' hosts.txt",
    "hints": [
      "Use s/localhost/127.0.0.1/gI",
      "Command: sed 's/localhost/127.0.0.1/gI' hosts.txt"
    ],
    "solution": "sed 's/localhost/127.0.0.1/gI' hosts.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/localhost\\/127\\.0\\.0\\.1\\/(gI|Ig)['\\\"]\\s+hosts\\.txt"
    ],
    "setup_files": {
      "hosts.txt": "LocalHost\nlocalhost\nLOCALHOST\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-30",
    "category": "Sed Stream Editing",
    "title": "Append Line After Pattern Match with 'a\\'",
    "difficulty": "Intermediate",
    "description": "Append a new line 'Include conf.modules.d/*.conf' after the line matching '# Load modules' in httpd.conf.",
    "objective": "sed '/# Load modules/a\\Include conf.modules.d/*.conf' httpd.conf",
    "hints": [
      "Use /pattern/a\\text",
      "Command: sed '/# Load modules/a\\Include conf.modules.d/*.conf' httpd.conf"
    ],
    "solution": "sed '/# Load modules/a\\Include conf.modules.d/*.conf' httpd.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/#\\s*Load modules\\/a.*Include.*conf['\\\"]\\s+httpd\\.conf"
    ],
    "setup_files": {
      "httpd.conf": "# Load modules\nServerRoot /etc/httpd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-26",
    "category": "Awk Processing",
    "title": "Print Lines Where Field Count NF Greater Than 3",
    "difficulty": "Easy",
    "description": "Print only lines from inventory.txt that have more than 3 whitespace-separated columns.",
    "objective": "awk 'NF > 3' inventory.txt",
    "hints": [
      "Condition: NF > 3",
      "Command: awk 'NF > 3' inventory.txt"
    ],
    "solution": "awk 'NF > 3' inventory.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]NF\\s*>\\s*3['\\\"]\\s+inventory\\.txt"
    ],
    "setup_files": {
      "inventory.txt": "a b c\na b c d\na b\na b c d e\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-27",
    "category": "Awk Processing",
    "title": "Prefix Every Line with Line Number (NR)",
    "difficulty": "Easy",
    "description": "Print line numbers alongside each line from data.txt using awk '{print NR, $0}'.",
    "objective": "awk '{print NR, $0}' data.txt",
    "hints": [
      "Use NR and $0",
      "Command: awk '{print NR, $0}' data.txt"
    ],
    "solution": "awk '{print NR, $0}' data.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{\\s*print\\s+NR\\s*,\\s*\\$0\\s*\\}['\\\"]\\s+data\\.txt"
    ],
    "setup_files": {
      "data.txt": "first\nsecond\nthird\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-28",
    "category": "Awk Processing",
    "title": "Calculate Column Average in END Block",
    "difficulty": "Intermediate",
    "description": "Calculate the arithmetic average of numeric column 2 in scores.txt and print the result.",
    "objective": "awk '{sum += $2} END {print sum/NR}' scores.txt",
    "hints": [
      "Accumulate sum += $2 and in END print sum/NR",
      "Command: awk '{sum += $2} END {print sum/NR}' scores.txt"
    ],
    "solution": "awk '{sum += $2} END {print sum/NR}' scores.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{\\s*sum\\s*\\+=\\s*\\$2\\s*\\}\\s*END\\s*\\{\\s*print\\s+sum\\s*\\/\\s*NR\\s*\\}['\\\"]\\s+scores\\.txt"
    ],
    "setup_files": {
      "scores.txt": "Alice 90\nBob 80\nCharlie 100\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-29",
    "category": "Awk Processing",
    "title": "Filter Lines Exceeding 80 Characters",
    "difficulty": "Easy",
    "description": "Find and display all lines in source.c longer than 80 characters using awk 'length($0) > 80'.",
    "objective": "awk 'length($0) > 80' source.c",
    "hints": [
      "Use length($0) > 80",
      "Command: awk 'length($0) > 80' source.c"
    ],
    "solution": "awk 'length($0) > 80' source.c",
    "accepted_regex": [
      "awk\\s+['\\\"]length(\\(\\$0\\))?\\s*>\\s*80['\\\"]\\s+source\\.c"
    ],
    "setup_files": {
      "source.c": "int main() { return 0; }\n/* aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa */\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-30",
    "category": "Awk Processing",
    "title": "Extract the Last Column Dynamically with $NF",
    "difficulty": "Easy",
    "description": "Extract and print the final column of each line in logs.txt regardless of how many columns exist.",
    "objective": "awk '{print $NF}' logs.txt",
    "hints": [
      "Use $NF for the last column",
      "Command: awk '{print $NF}' logs.txt"
    ],
    "solution": "awk '{print $NF}' logs.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\{\\s*print\\s+\\$NF\\s*\\}['\\\"]\\s+logs\\.txt"
    ],
    "setup_files": {
      "logs.txt": "2026-10-09 GET 200 /index.html\n2026-10-09 POST /login\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-26",
    "category": "Pipes & Redirections",
    "title": "Merge Two Files Horizontally with paste -d",
    "difficulty": "Easy",
    "description": "Combine names.txt and ages.txt side by side, delimited by a comma (paste -d',').",
    "objective": "paste -d',' names.txt ages.txt",
    "hints": [
      "Use paste -d',' names.txt ages.txt",
      "Command: paste -d',' names.txt ages.txt"
    ],
    "solution": "paste -d',' names.txt ages.txt",
    "accepted_regex": [
      "paste\\s+-d['\\\"]?,['\\\"]?\\s+names\\.txt\\s+ages\\.txt"
    ],
    "setup_files": {
      "names.txt": "Alice\nBob\n",
      "ages.txt": "30\n25\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-27",
    "category": "Pipes & Redirections",
    "title": "Join Files by Common Leading Field with join",
    "difficulty": "Intermediate",
    "description": "Join sorted files id_names.txt and id_dept.txt on their shared first field (ID).",
    "objective": "join id_names.txt id_dept.txt",
    "hints": [
      "Use join id_names.txt id_dept.txt",
      "Command: join id_names.txt id_dept.txt"
    ],
    "solution": "join id_names.txt id_dept.txt",
    "accepted_regex": [
      "join\\s+id_names\\.txt\\s+id_dept\\.txt"
    ],
    "setup_files": {
      "id_names.txt": "101 Alice\n102 Bob\n",
      "id_dept.txt": "101 Engineering\n102 Finance\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-28",
    "category": "Pipes & Redirections",
    "title": "Silence Both Output and Errors with &> /dev/null",
    "difficulty": "Easy",
    "description": "Execute ping -c 1 8.8.8.8 and discard both standard output and standard error completely.",
    "objective": "ping -c 1 8.8.8.8 &> /dev/null",
    "hints": [
      "Redirect with &> /dev/null or > /dev/null 2>&1",
      "Command: ping -c 1 8.8.8.8 &> /dev/null"
    ],
    "solution": "ping -c 1 8.8.8.8 &> /dev/null",
    "accepted_regex": [
      "ping\\s+-c\\s*1\\s+8\\.8\\.8\\.8\\s+(&>\\s*\\/dev\\/null|>\\s*\\/dev\\/null\\s+2>&1)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-29",
    "category": "Pipes & Redirections",
    "title": "Append String to File as Superuser with tee -a",
    "difficulty": "Intermediate",
    "description": "Echo 'nameserver 1.1.1.1' and append it to /etc/resolv.conf using sudo tee -a.",
    "objective": "echo 'nameserver 1.1.1.1' | sudo tee -a /etc/resolv.conf",
    "hints": [
      "Pipe echo to tee -a /etc/resolv.conf",
      "Command: echo 'nameserver 1.1.1.1' | sudo tee -a /etc/resolv.conf"
    ],
    "solution": "echo 'nameserver 1.1.1.1' | sudo tee -a /etc/resolv.conf",
    "accepted_regex": [
      "echo\\s+['\\\"]nameserver 1\\.1\\.1\\.1['\\\"]\\s*\\|\\s*(sudo\\s+)?tee\\s+-a\\s+\\/etc\\/resolv\\.conf"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-30",
    "category": "Pipes & Redirections",
    "title": "Safe Whitespace Pipe with find -print0 | xargs -0",
    "difficulty": "Advanced",
    "description": "Safely delete all .tmp files in /var/tmp even if names contain spaces or newlines using find -print0 and xargs -0 rm -f.",
    "objective": "find /var/tmp -type f -name '*.tmp' -print0 | xargs -0 rm -f",
    "hints": [
      "Use find -print0 | xargs -0 rm -f",
      "Command: find /var/tmp -type f -name '*.tmp' -print0 | xargs -0 rm -f"
    ],
    "solution": "find /var/tmp -type f -name '*.tmp' -print0 | xargs -0 rm -f",
    "accepted_regex": [
      "find\\s+\\/var\\/tmp\\s+.*-print0\\s*\\|\\s*xargs\\s+-0\\s+rm\\s+-f"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-41",
    "category": "Service Management",
    "title": "Analyze System Bootup Performance Times",
    "difficulty": "Easy",
    "description": "Analyze system boot time performance breakdown using systemd-analyze.",
    "objective": "systemd-analyze",
    "hints": [
      "Command: systemd-analyze"
    ],
    "solution": "systemd-analyze",
    "accepted_regex": [
      "systemd-analyze(\\s+time)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-42",
    "category": "Service Management",
    "title": "List Slowest Units During Boot Sequence",
    "difficulty": "Intermediate",
    "description": "Print the top slowest systemd services during the boot initialization.",
    "objective": "systemd-analyze blame",
    "hints": [
      "Command: systemd-analyze blame"
    ],
    "solution": "systemd-analyze blame",
    "accepted_regex": [
      "systemd-analyze\\s+blame"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-43",
    "category": "Service Management",
    "title": "Inspect Critical Chain in System Startup",
    "difficulty": "Intermediate",
    "description": "Display the time-critical chain of systemd dependencies that determine boot duration.",
    "objective": "systemd-analyze critical-chain",
    "hints": [
      "Command: systemd-analyze critical-chain"
    ],
    "solution": "systemd-analyze critical-chain",
    "accepted_regex": [
      "systemd-analyze\\s+critical-chain"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-44",
    "category": "Service Management",
    "title": "Create Drop-In Configuration Override with Edit",
    "difficulty": "Intermediate",
    "description": "Open an editor to create a drop-in override configuration directory and file for httpd.",
    "objective": "systemctl edit httpd",
    "hints": [
      "Command: systemctl edit httpd"
    ],
    "solution": "systemctl edit httpd",
    "accepted_regex": [
      "systemctl\\s+edit\\s+httpd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-45",
    "category": "Service Management",
    "title": "Full Replacement Override via systemctl edit --full",
    "difficulty": "Advanced",
    "description": "Create a full copy replacement unit file in /etc/systemd/system for editing crond.",
    "objective": "systemctl edit --full crond",
    "hints": [
      "Command: systemctl edit --full crond"
    ],
    "solution": "systemctl edit --full crond",
    "accepted_regex": [
      "systemctl\\s+edit\\s+--full\\s+crond(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-46",
    "category": "Service Management",
    "title": "Revert Overridden Unit to Vendor Defaults",
    "difficulty": "Intermediate",
    "description": "Revert all drop-ins and overrides for unit 'httpd' back to vendor settings.",
    "objective": "systemctl revert httpd",
    "hints": [
      "Command: systemctl revert httpd"
    ],
    "solution": "systemctl revert httpd",
    "accepted_regex": [
      "systemctl\\s+revert\\s+httpd(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-47",
    "category": "Service Management",
    "title": "List All Active Mount Units",
    "difficulty": "Easy",
    "description": "List all active filesystem mount units managed by systemd.",
    "objective": "systemctl list-units --type=mount",
    "hints": [
      "Command: systemctl list-units --type=mount"
    ],
    "solution": "systemctl list-units --type=mount",
    "accepted_regex": [
      "systemctl\\s+list-units\\s+--type=mount"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-48",
    "category": "Service Management",
    "title": "List All Active Path Monitoring Units",
    "difficulty": "Easy",
    "description": "Display active .path units that monitor files and directories for changes.",
    "objective": "systemctl list-units --type=path",
    "hints": [
      "Command: systemctl list-units --type=path"
    ],
    "solution": "systemctl list-units --type=path",
    "accepted_regex": [
      "systemctl\\s+list-units\\s+--type=path"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-49",
    "category": "Service Management",
    "title": "Display Main PID of Running Service",
    "difficulty": "Intermediate",
    "description": "Query only the numeric MainPID attribute of service sshd.",
    "objective": "systemctl show -p MainPID --value sshd",
    "hints": [
      "Command: systemctl show -p MainPID --value sshd"
    ],
    "solution": "systemctl show -p MainPID --value sshd",
    "accepted_regex": [
      "systemctl\\s+show\\s+(-p\\s+MainPID\\s+--value|--value\\s+-p\\s+MainPID)\\s+sshd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-50",
    "category": "Service Management",
    "title": "Display Active State Property Only",
    "difficulty": "Intermediate",
    "description": "Query only the active state value (active/inactive/failed) of chronyd with --value.",
    "objective": "systemctl show -p ActiveState --value chronyd",
    "hints": [
      "Command: systemctl show -p ActiveState --value chronyd"
    ],
    "solution": "systemctl show -p ActiveState --value chronyd",
    "accepted_regex": [
      "systemctl\\s+show\\s+(-p\\s+ActiveState\\s+--value|--value\\s+-p\\s+ActiveState)\\s+chronyd"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-51",
    "category": "Service Management",
    "title": "Start Ephemeral Transient Unit with systemd-run",
    "difficulty": "Advanced",
    "description": "Run a transient service executing '/bin/sleep 60' in the background.",
    "objective": "systemd-run /bin/sleep 60",
    "hints": [
      "Command: systemd-run /bin/sleep 60"
    ],
    "solution": "systemd-run /bin/sleep 60",
    "accepted_regex": [
      "systemd-run\\s+(\\/bin\\/sleep|sleep)\\s+60"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-52",
    "category": "Service Management",
    "title": "Run Transient Unit with Memory Limit",
    "difficulty": "Advanced",
    "description": "Execute '/usr/bin/python3 script.py' with systemd-run enforcing a 256MB memory cap.",
    "objective": "systemd-run -p MemoryMax=256M python3 script.py",
    "hints": [
      "Command: systemd-run -p MemoryMax=256M python3 script.py"
    ],
    "solution": "systemd-run -p MemoryMax=256M python3 script.py",
    "accepted_regex": [
      "systemd-run\\s+(-p|--property=)MemoryMax=256M\\s+python3\\s+script\\.py"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-53",
    "category": "Service Management",
    "title": "Inspect Control Group Hierarchy with systemd-cgls",
    "difficulty": "Easy",
    "description": "View the recursively structured control group hierarchy tree.",
    "objective": "systemd-cgls",
    "hints": [
      "Command: systemd-cgls"
    ],
    "solution": "systemd-cgls",
    "accepted_regex": [
      "systemd-cgls"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-54",
    "category": "Service Management",
    "title": "Monitor Real-Time Cgroup Resource Usage with systemd-cgtop",
    "difficulty": "Easy",
    "description": "Launch top-like real-time resource monitor for control group slices and services in batch/one iteration.",
    "objective": "systemd-cgtop -n 1",
    "hints": [
      "Command: systemd-cgtop -n 1"
    ],
    "solution": "systemd-cgtop -n 1",
    "accepted_regex": [
      "systemd-cgtop(\\s+-n\\s+1)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-55",
    "category": "Service Management",
    "title": "Filter Journal by Syslog Identifier Tag",
    "difficulty": "Intermediate",
    "description": "Query journal logs matching SYSLOG_IDENTIFIER='kernel' or 'sudo'.",
    "objective": "journalctl -t sudo -n 20",
    "hints": [
      "Command: journalctl -t sudo -n 20"
    ],
    "solution": "journalctl -t sudo -n 20",
    "accepted_regex": [
      "journalctl\\s+.*-t\\s+sudo.*-n\\s+20",
      "journalctl\\s+.*-n\\s+20.*-t\\s+sudo"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-56",
    "category": "Service Management",
    "title": "Verify Journal Catalog Entry Explanations",
    "difficulty": "Intermediate",
    "description": "Display journal entries with metadata explanations attached using catalog mode (-x).",
    "objective": "journalctl -u sshd -x -n 10",
    "hints": [
      "Command: journalctl -u sshd -x -n 10"
    ],
    "solution": "journalctl -u sshd -x -n 10",
    "accepted_regex": [
      "journalctl\\s+.*-u\\s+sshd.*-x.*-n\\s+10",
      "journalctl\\s+.*-x.*-u\\s+sshd.*-n\\s+10"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-57",
    "category": "Service Management",
    "title": "Sync and Flush Journal from Memory to Persistent Disk",
    "difficulty": "Intermediate",
    "description": "Instruct journald to write and flush all volatile in-memory log data to disk.",
    "objective": "journalctl --flush",
    "hints": [
      "Command: journalctl --flush"
    ],
    "solution": "journalctl --flush",
    "accepted_regex": [
      "journalctl\\s+--flush"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-58",
    "category": "Service Management",
    "title": "Rotate Systemd Journal Files Immediately",
    "difficulty": "Intermediate",
    "description": "Force journald to immediately rotate active journal log files.",
    "objective": "journalctl --rotate",
    "hints": [
      "Command: journalctl --rotate"
    ],
    "solution": "journalctl --rotate",
    "accepted_regex": [
      "journalctl\\s+--rotate"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-59",
    "category": "Service Management",
    "title": "Verify Journal File Integrity",
    "difficulty": "Intermediate",
    "description": "Audit the internal cryptographic and structural integrity of journal files.",
    "objective": "journalctl --verify",
    "hints": [
      "Command: journalctl --verify"
    ],
    "solution": "journalctl --verify",
    "accepted_regex": [
      "journalctl\\s+--verify"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-60",
    "category": "Service Management",
    "title": "List All Recorded System Boot Identifiers",
    "difficulty": "Easy",
    "description": "List all boots tracked by journalctl with their index numbers and boot IDs.",
    "objective": "journalctl --list-boots",
    "hints": [
      "Command: journalctl --list-boots"
    ],
    "solution": "journalctl --list-boots",
    "accepted_regex": [
      "journalctl\\s+--list-boots"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-61",
    "category": "Service Management",
    "title": "Filter Journal Logs by Transport Mechanism",
    "difficulty": "Intermediate",
    "description": "Filter journalctl to only show messages coming via the stdout transport.",
    "objective": "journalctl _TRANSPORT=stdout -n 30",
    "hints": [
      "Command: journalctl _TRANSPORT=stdout -n 30"
    ],
    "solution": "journalctl _TRANSPORT=stdout -n 30",
    "accepted_regex": [
      "journalctl\\s+.*_TRANSPORT=stdout.*-n\\s+30"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-62",
    "category": "Service Management",
    "title": "Check Status of Target Unit",
    "difficulty": "Easy",
    "description": "Check current state of 'multi-user.target' using systemctl status.",
    "objective": "systemctl status multi-user.target",
    "hints": [
      "Command: systemctl status multi-user.target"
    ],
    "solution": "systemctl status multi-user.target",
    "accepted_regex": [
      "systemctl\\s+status\\s+multi-user\\.target"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-63",
    "category": "Service Management",
    "title": "List Active Target Units",
    "difficulty": "Easy",
    "description": "Display all systemd target units currently active on the host.",
    "objective": "systemctl list-units --type=target",
    "hints": [
      "Command: systemctl list-units --type=target"
    ],
    "solution": "systemctl list-units --type=target",
    "accepted_regex": [
      "systemctl\\s+list-units\\s+--type=target"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-64",
    "category": "Service Management",
    "title": "Set Runtime Cgroup Memory Limit on Live Service",
    "difficulty": "Advanced",
    "description": "Set MemoryMax property on running unit httpd to 500M dynamically.",
    "objective": "systemctl set-property httpd MemoryMax=500M",
    "hints": [
      "Command: systemctl set-property httpd MemoryMax=500M"
    ],
    "solution": "systemctl set-property httpd MemoryMax=500M",
    "accepted_regex": [
      "systemctl\\s+set-property\\s+httpd(\\.service)?\\s+MemoryMax=500M"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-65",
    "category": "Service Management",
    "title": "Set Runtime CPUQuota on Live Service",
    "difficulty": "Advanced",
    "description": "Throttle service nginx to use at most 50% CPU allocation via CPUQuota.",
    "objective": "systemctl set-property nginx CPUQuota=50%",
    "hints": [
      "Command: systemctl set-property nginx CPUQuota=50%"
    ],
    "solution": "systemctl set-property nginx CPUQuota=50%",
    "accepted_regex": [
      "systemctl\\s+set-property\\s+nginx(\\.service)?\\s+CPUQuota=50%"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-66",
    "category": "Service Management",
    "title": "Display Current System Date, Time, and NTP Synchronization",
    "difficulty": "Easy",
    "description": "Display system date, timezone, and network time synchronization state with timedatectl.",
    "objective": "timedatectl status",
    "hints": [
      "Command: timedatectl status"
    ],
    "solution": "timedatectl status",
    "accepted_regex": [
      "timedatectl(\\s+status)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-67",
    "category": "Service Management",
    "title": "Set System Timezone to America/New_York",
    "difficulty": "Intermediate",
    "description": "Configure the local system timezone to 'America/New_York' with timedatectl.",
    "objective": "timedatectl set-timezone America/New_York",
    "hints": [
      "Command: timedatectl set-timezone America/New_York"
    ],
    "solution": "timedatectl set-timezone America/New_York",
    "accepted_regex": [
      "timedatectl\\s+set-timezone\\s+America\\/New_York"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-68",
    "category": "Service Management",
    "title": "Enable NTP Time Synchronization",
    "difficulty": "Easy",
    "description": "Enable automatic network time synchronization (chrony/systemd-timesyncd) with timedatectl.",
    "objective": "timedatectl set-ntp true",
    "hints": [
      "Command: timedatectl set-ntp true"
    ],
    "solution": "timedatectl set-ntp true",
    "accepted_regex": [
      "timedatectl\\s+set-ntp\\s+(true|1|on)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-69",
    "category": "Service Management",
    "title": "Inspect Hostname and OS Architecture Details",
    "difficulty": "Easy",
    "description": "View static/pretty hostname, kernel version, and architecture using hostnamectl.",
    "objective": "hostnamectl status",
    "hints": [
      "Command: hostnamectl status"
    ],
    "solution": "hostnamectl status",
    "accepted_regex": [
      "hostnamectl(\\s+status)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-70",
    "category": "Service Management",
    "title": "Set Static System FQDN Hostname",
    "difficulty": "Intermediate",
    "description": "Configure system static hostname to 'server1.example.com' with hostnamectl.",
    "objective": "hostnamectl set-hostname server1.example.com",
    "hints": [
      "Command: hostnamectl set-hostname server1.example.com"
    ],
    "solution": "hostnamectl set-hostname server1.example.com",
    "accepted_regex": [
      "hostnamectl\\s+set-hostname\\s+server1\\.example\\.com"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-71",
    "category": "Service Management",
    "title": "Audit Active User Logins and Sessions with loginctl",
    "difficulty": "Easy",
    "description": "List all active user sessions and logged-in accounts with loginctl.",
    "objective": "loginctl list-sessions",
    "hints": [
      "Command: loginctl list-sessions"
    ],
    "solution": "loginctl list-sessions",
    "accepted_regex": [
      "loginctl\\s+list-sessions"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-72",
    "category": "Service Management",
    "title": "Inspect Specific User Session Properties",
    "difficulty": "Intermediate",
    "description": "Inspect details of session '2' including state and idle status using loginctl session-status.",
    "objective": "loginctl session-status 2",
    "hints": [
      "Command: loginctl session-status 2"
    ],
    "solution": "loginctl session-status 2",
    "accepted_regex": [
      "loginctl\\s+session-status\\s+2"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-73",
    "category": "Service Management",
    "title": "Enable User Lingering for Background Processes",
    "difficulty": "Intermediate",
    "description": "Allow non-root user 'ansible' to keep systemd user services running when logged out.",
    "objective": "loginctl enable-linger ansible",
    "hints": [
      "Command: loginctl enable-linger ansible"
    ],
    "solution": "loginctl enable-linger ansible",
    "accepted_regex": [
      "loginctl\\s+enable-linger\\s+ansible"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-74",
    "category": "Service Management",
    "title": "Trigger Systemd Timer Execution Manually",
    "difficulty": "Intermediate",
    "description": "Trigger the target service 'dnf-makecache.service' of a timer immediately without waiting for schedule.",
    "objective": "systemctl start dnf-makecache.service",
    "hints": [
      "Command: systemctl start dnf-makecache.service"
    ],
    "solution": "systemctl start dnf-makecache.service",
    "accepted_regex": [
      "systemctl\\s+start\\s+dnf-makecache(\\.service)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-75",
    "category": "Service Management",
    "title": "Display systemd Environment Variables",
    "difficulty": "Easy",
    "description": "Print the environment block configured for the systemd PID 1 manager.",
    "objective": "systemctl show-environment",
    "hints": [
      "Command: systemctl show-environment"
    ],
    "solution": "systemctl show-environment",
    "accepted_regex": [
      "systemctl\\s+show-environment"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-76",
    "category": "Service Management",
    "title": "Set Global Environment Variable for systemd Services",
    "difficulty": "Intermediate",
    "description": "Import environment variable 'PROXY_URL=http://proxy:8080' into the systemd manager.",
    "objective": "systemctl set-environment PROXY_URL=http://proxy:8080",
    "hints": [
      "Command: systemctl set-environment PROXY_URL=http://proxy:8080"
    ],
    "solution": "systemctl set-environment PROXY_URL=http://proxy:8080",
    "accepted_regex": [
      "systemctl\\s+set-environment\\s+PROXY_URL=http:\\/\\/proxy:8080"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-77",
    "category": "Service Management",
    "title": "Re-execute Systemd Daemon Process",
    "difficulty": "Advanced",
    "description": "Re-execute the systemd system manager process (re-executes PID 1) without rebooting.",
    "objective": "systemctl daemon-reexec",
    "hints": [
      "Command: systemctl daemon-reexec"
    ],
    "solution": "systemctl daemon-reexec",
    "accepted_regex": [
      "systemctl\\s+daemon-reexec"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-78",
    "category": "Service Management",
    "title": "Emergency Reboot Using systemctl",
    "difficulty": "Intermediate",
    "description": "Force immediate emergency reboot without unmounting filesystems or contacting init.",
    "objective": "systemctl reboot --force --force",
    "hints": [
      "Command: systemctl reboot --force --force"
    ],
    "solution": "systemctl reboot --force --force",
    "accepted_regex": [
      "systemctl\\s+reboot\\s+--force\\s+--force",
      "systemctl\\s+-ff\\s+reboot"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-79",
    "category": "Service Management",
    "title": "Switch System into Rescue Target Mode",
    "difficulty": "Intermediate",
    "description": "Isolate single-user maintenance mode (rescue.target) for emergency repair.",
    "objective": "systemctl isolate rescue.target",
    "hints": [
      "Command: systemctl isolate rescue.target"
    ],
    "solution": "systemctl isolate rescue.target",
    "accepted_regex": [
      "systemctl\\s+isolate\\s+rescue\\.target",
      "systemctl\\s+rescue"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sys-80",
    "category": "Service Management",
    "title": "Switch System into Emergency Minimal Shell",
    "difficulty": "Intermediate",
    "description": "Isolate the barest minimal emergency mode (emergency.target).",
    "objective": "systemctl isolate emergency.target",
    "hints": [
      "Command: systemctl isolate emergency.target"
    ],
    "solution": "systemctl isolate emergency.target",
    "accepted_regex": [
      "systemctl\\s+isolate\\s+emergency\\.target",
      "systemctl\\s+emergency"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-41",
    "category": "Firewall & Network",
    "title": "Set Default Zone Gateway in NetworkManager",
    "difficulty": "Intermediate",
    "description": "Assign default IPv4 gateway 192.168.1.1 to connection profile 'ens192'.",
    "objective": "nmcli con mod ens192 ipv4.gateway 192.168.1.1",
    "hints": [
      "Command: nmcli con mod ens192 ipv4.gateway 192.168.1.1"
    ],
    "solution": "nmcli con mod ens192 ipv4.gateway 192.168.1.1",
    "accepted_regex": [
      "nmcli\\s+con\\s+mod(ify)?\\s+ens192\\s+ipv4\\.gateway\\s+192\\.168\\.1\\.1"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-42",
    "category": "Firewall & Network",
    "title": "Delete NetworkManager Connection Profile",
    "difficulty": "Easy",
    "description": "Delete existing connection profile named 'old-backup-lan'.",
    "objective": "nmcli con del old-backup-lan",
    "hints": [
      "Command: nmcli con del old-backup-lan"
    ],
    "solution": "nmcli con del old-backup-lan",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+(del|delete)\\s+old-backup-lan"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-43",
    "category": "Firewall & Network",
    "title": "Deactivate Network Connection Temporarily",
    "difficulty": "Easy",
    "description": "Take down network connection 'ens192' using nmcli.",
    "objective": "nmcli con down ens192",
    "hints": [
      "Command: nmcli con down ens192"
    ],
    "solution": "nmcli con down ens192",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+down\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-44",
    "category": "Firewall & Network",
    "title": "Configure Connection Autoconnect Behavior",
    "difficulty": "Easy",
    "description": "Enable automatic connection on boot for profile 'ens192'.",
    "objective": "nmcli con mod ens192 connection.autoconnect yes",
    "hints": [
      "Command: nmcli con mod ens192 connection.autoconnect yes"
    ],
    "solution": "nmcli con mod ens192 connection.autoconnect yes",
    "accepted_regex": [
      "nmcli\\s+con\\s+mod(ify)?\\s+ens192\\s+connection\\.autoconnect\\s+(yes|true)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-45",
    "category": "Firewall & Network",
    "title": "Inspect Detailed Connection Settings with nmcli",
    "difficulty": "Intermediate",
    "description": "Display all configured keys and values for connection profile 'ens192'.",
    "objective": "nmcli con show ens192",
    "hints": [
      "Command: nmcli con show ens192"
    ],
    "solution": "nmcli con show ens192",
    "accepted_regex": [
      "nmcli\\s+(con|connection)\\s+show\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-46",
    "category": "Firewall & Network",
    "title": "Create Static Static Route with nmcli",
    "difficulty": "Advanced",
    "description": "Add static route for 10.50.0.0/16 via gateway 192.168.1.254 to profile 'ens192'.",
    "objective": "nmcli con mod ens192 +ipv4.routes \"10.50.0.0/16 192.168.1.254\"",
    "hints": [
      "Command: nmcli con mod ens192 +ipv4.routes \"10.50.0.0/16 192.168.1.254\""
    ],
    "solution": "nmcli con mod ens192 +ipv4.routes \"10.50.0.0/16 192.168.1.254\"",
    "accepted_regex": [
      "nmcli\\s+con\\s+mod(ify)?\\s+ens192\\s+\\+?ipv4\\.routes\\s+[\\\"']10\\.50\\.0\\.0\\/16\\s+192\\.168\\.1\\.254[\\\"']"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-47",
    "category": "Firewall & Network",
    "title": "Create Network Team/Bond Master Interface",
    "difficulty": "Advanced",
    "description": "Create a team connection profile named 'team0' with runner activebackup.",
    "objective": "nmcli con add type team con-name team0 ifname team0 config '{\"runner\": {\"name\": \"activebackup\"}}'",
    "hints": [
      "Command: nmcli con add type team con-name team0 ifname team0 config '{\"runner\": {\"name\": \"activebackup\"}}'"
    ],
    "solution": "nmcli con add type team con-name team0 ifname team0 config '{\"runner\": {\"name\": \"activebackup\"}}'",
    "accepted_regex": [
      "nmcli\\s+con\\s+add\\s+type\\s+team\\s+con-name\\s+team0\\s+ifname\\s+team0"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-48",
    "category": "Firewall & Network",
    "title": "Add Slave Port to Network Team Profile",
    "difficulty": "Advanced",
    "description": "Add device ens224 as a team-slave port to team master 'team0'.",
    "objective": "nmcli con add type team-slave con-name team0-port1 ifname ens224 master team0",
    "hints": [
      "Command: nmcli con add type team-slave con-name team0-port1 ifname ens224 master team0"
    ],
    "solution": "nmcli con add type team-slave con-name team0-port1 ifname ens224 master team0",
    "accepted_regex": [
      "nmcli\\s+con\\s+add\\s+type\\s+team-slave\\s+con-name\\s+team0-port1\\s+ifname\\s+ens224\\s+master\\s+team0"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-49",
    "category": "Firewall & Network",
    "title": "Permanently Block IP Subnet with Firewalld Rich Rule",
    "difficulty": "Advanced",
    "description": "Drop all incoming traffic from abusive subnet 198.51.100.0/24 in public zone permanently.",
    "objective": "firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"198.51.100.0/24\" drop'",
    "hints": [
      "Command: firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"198.51.100.0/24\" drop'"
    ],
    "solution": "firewall-cmd --permanent --add-rich-rule='rule family=\"ipv4\" source address=\"198.51.100.0/24\" drop'",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-rich-rule=.*198\\.51\\.100\\.0\\/24.*drop"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-50",
    "category": "Firewall & Network",
    "title": "Rate Limit Logged SSH Connections with Rich Rule",
    "difficulty": "Advanced",
    "description": "Limit SSH connection rate to 3/minute with prefix logging using rich rule.",
    "objective": "firewall-cmd --permanent --add-rich-rule='rule service name=\"ssh\" log prefix=\"ssh_flood: \" level=\"info\" limit value=\"3/m\" accept'",
    "hints": [
      "Command: firewall-cmd --permanent --add-rich-rule='rule service name=\"ssh\" log prefix=\"ssh_flood: \" level=\"info\" limit value=\"3/m\" accept'"
    ],
    "solution": "firewall-cmd --permanent --add-rich-rule='rule service name=\"ssh\" log prefix=\"ssh_flood: \" level=\"info\" limit value=\"3/m\" accept'",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--add-rich-rule=.*limit\\s+value=[\\\"']3\\/m[\\\"']"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-51",
    "category": "Firewall & Network",
    "title": "Create Custom Firewalld Service Definition",
    "difficulty": "Intermediate",
    "description": "Create a new permanent empty service named 'myapp' in firewalld.",
    "objective": "firewall-cmd --permanent --new-service=myapp",
    "hints": [
      "Command: firewall-cmd --permanent --new-service=myapp"
    ],
    "solution": "firewall-cmd --permanent --new-service=myapp",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--new-service=myapp"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-52",
    "category": "Firewall & Network",
    "title": "Assign Port to Custom Firewalld Service",
    "difficulty": "Intermediate",
    "description": "Add TCP port 9000 to custom service 'myapp' permanently.",
    "objective": "firewall-cmd --permanent --service=myapp --add-port=9000/tcp",
    "hints": [
      "Command: firewall-cmd --permanent --service=myapp --add-port=9000/tcp"
    ],
    "solution": "firewall-cmd --permanent --service=myapp --add-port=9000/tcp",
    "accepted_regex": [
      "firewall-cmd\\s+.*--permanent.*--service=myapp.*--add-port=9000\\/tcp"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-53",
    "category": "Firewall & Network",
    "title": "Enable Panic Mode to Drop All Network Traffic",
    "difficulty": "Advanced",
    "description": "Immediately cut all incoming and outgoing network traffic with emergency panic mode.",
    "objective": "firewall-cmd --panic-on",
    "hints": [
      "Command: firewall-cmd --panic-on"
    ],
    "solution": "firewall-cmd --panic-on",
    "accepted_regex": [
      "firewall-cmd\\s+--panic-on"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-54",
    "category": "Firewall & Network",
    "title": "Disable Panic Mode to Restore Connectivity",
    "difficulty": "Easy",
    "description": "Disable emergency panic mode to restore normal network packet processing.",
    "objective": "firewall-cmd --panic-off",
    "hints": [
      "Command: firewall-cmd --panic-off"
    ],
    "solution": "firewall-cmd --panic-off",
    "accepted_regex": [
      "firewall-cmd\\s+--panic-off"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-55",
    "category": "Firewall & Network",
    "title": "Inspect Persistent SELinux Port Labels",
    "difficulty": "Intermediate",
    "description": "List all network port definitions registered in SELinux policy using semanage.",
    "objective": "semanage port -l",
    "hints": [
      "Command: semanage port -l"
    ],
    "solution": "semanage port -l",
    "accepted_regex": [
      "semanage\\s+port\\s+-l"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-56",
    "category": "Firewall & Network",
    "title": "Assign Custom TCP Port to HTTP SELinux Context",
    "difficulty": "Advanced",
    "description": "Add port 8088/tcp to http_port_t SELinux type policy permanently.",
    "objective": "semanage port -a -t http_port_t -p tcp 8088",
    "hints": [
      "Command: semanage port -a -t http_port_t -p tcp 8088"
    ],
    "solution": "semanage port -a -t http_port_t -p tcp 8088",
    "accepted_regex": [
      "semanage\\s+port\\s+-a\\s+-t\\s+http_port_t\\s+-p\\s+tcp\\s+8088"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-57",
    "category": "Firewall & Network",
    "title": "Assign Custom Directory Context in SELinux Policy",
    "difficulty": "Advanced",
    "description": "Add permanent file context rule for '/custom_web(/.*)?' as httpd_sys_content_t.",
    "objective": "semanage fcontext -a -t httpd_sys_content_t \"/custom_web(/.*)?\"",
    "hints": [
      "Command: semanage fcontext -a -t httpd_sys_content_t \"/custom_web(/.*)?\""
    ],
    "solution": "semanage fcontext -a -t httpd_sys_content_t \"/custom_web(/.*)?\"",
    "accepted_regex": [
      "semanage\\s+fcontext\\s+-a\\s+-t\\s+httpd_sys_content_t\\s+[\\\"']\\/custom_web\\(\\/\\.\\*\\)\\?[\\\"']"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-58",
    "category": "Firewall & Network",
    "title": "Search SELinux Audit Denials with ausearch",
    "difficulty": "Intermediate",
    "description": "Search audit.log for recent SELinux AVC denials in past 10 minutes with ausearch.",
    "objective": "ausearch -m avc -ts recent",
    "hints": [
      "Command: ausearch -m avc -ts recent"
    ],
    "solution": "ausearch -m avc -ts recent",
    "accepted_regex": [
      "ausearch\\s+-m\\s+avc(\\s+-ts\\s+recent)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-59",
    "category": "Firewall & Network",
    "title": "Analyze SELinux Denials and Suggest Fixes",
    "difficulty": "Intermediate",
    "description": "Translate raw audit logs into human-readable suggestions using audit2why.",
    "objective": "audit2why < /var/log/audit/audit.log",
    "hints": [
      "Command: audit2why < /var/log/audit/audit.log or ausearch -m avc | audit2why"
    ],
    "solution": "ausearch -m avc | audit2why",
    "accepted_regex": [
      "(ausearch\\s+-m\\s+avc\\s*\\|\\s*audit2why|audit2why\\s*<\\s*\\/var\\/log\\/audit\\/audit\\.log)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-60",
    "category": "Firewall & Network",
    "title": "Display All Listening UDP Sockets with ss",
    "difficulty": "Easy",
    "description": "Display all listening UDP sockets with numeric addresses and process details.",
    "objective": "ss -ulpn",
    "hints": [
      "Command: ss -ulpn"
    ],
    "solution": "ss -ulpn",
    "accepted_regex": [
      "ss\\s+-(ulpn|unlp|lupn)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-61",
    "category": "Firewall & Network",
    "title": "Filter Established TCP Sockets to Specific Remote Host",
    "difficulty": "Intermediate",
    "description": "Display established TCP connections to destination 10.0.0.1 using ss filter syntax.",
    "objective": "ss -t state established dst 10.0.0.1",
    "hints": [
      "Command: ss -t state established dst 10.0.0.1"
    ],
    "solution": "ss -t state established dst 10.0.0.1",
    "accepted_regex": [
      "ss\\s+.*state\\s+established.*dst\\s+10\\.0\\.0\\.1"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-62",
    "category": "Firewall & Network",
    "title": "Display Socket Memory Buffer Sizes",
    "difficulty": "Intermediate",
    "description": "Show socket memory consumption (-m) and internal TCP info (-i) with ss.",
    "objective": "ss -tmi",
    "hints": [
      "Command: ss -tmi"
    ],
    "solution": "ss -tmi",
    "accepted_regex": [
      "ss\\s+-(tmi|tim)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-63",
    "category": "Firewall & Network",
    "title": "Add Temporary IP Address to Interface via iproute2",
    "difficulty": "Easy",
    "description": "Add secondary IP 10.0.0.99/24 to device ens192 using ip addr.",
    "objective": "ip addr add 10.0.0.99/24 dev ens192",
    "hints": [
      "Command: ip addr add 10.0.0.99/24 dev ens192"
    ],
    "solution": "ip addr add 10.0.0.99/24 dev ens192",
    "accepted_regex": [
      "ip\\s+(addr|a)\\s+add\\s+10\\.0\\.0\\.99\\/24\\s+dev\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-64",
    "category": "Firewall & Network",
    "title": "Delete Temporary IP Address from Interface",
    "difficulty": "Easy",
    "description": "Remove IP address 10.0.0.99/24 from interface ens192 using ip addr.",
    "objective": "ip addr del 10.0.0.99/24 dev ens192",
    "hints": [
      "Command: ip addr del 10.0.0.99/24 dev ens192"
    ],
    "solution": "ip addr del 10.0.0.99/24 dev ens192",
    "accepted_regex": [
      "ip\\s+(addr|a)\\s+del\\s+10\\.0\\.0\\.99\\/24\\s+dev\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-65",
    "category": "Firewall & Network",
    "title": "Set Interface Link State Up",
    "difficulty": "Easy",
    "description": "Bring network device ens224 link state UP using ip link.",
    "objective": "ip link set ens224 up",
    "hints": [
      "Command: ip link set ens224 up"
    ],
    "solution": "ip link set ens224 up",
    "accepted_regex": [
      "ip\\s+link\\s+set\\s+ens224\\s+up"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-66",
    "category": "Firewall & Network",
    "title": "Set Interface Link State Down",
    "difficulty": "Easy",
    "description": "Bring network device ens224 link state DOWN using ip link.",
    "objective": "ip link set ens224 down",
    "hints": [
      "Command: ip link set ens224 down"
    ],
    "solution": "ip link set ens224 down",
    "accepted_regex": [
      "ip\\s+link\\s+set\\s+ens224\\s+down"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-67",
    "category": "Firewall & Network",
    "title": "Flush ARP Neighbor Cache on Device",
    "difficulty": "Intermediate",
    "description": "Flush all ARP neighbor entries associated with device ens192.",
    "objective": "ip neigh flush dev ens192",
    "hints": [
      "Command: ip neigh flush dev ens192"
    ],
    "solution": "ip neigh flush dev ens192",
    "accepted_regex": [
      "ip\\s+neigh\\s+flush\\s+dev\\s+ens192"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-68",
    "category": "Firewall & Network",
    "title": "Inspect Packet Routing Policy Rules",
    "difficulty": "Intermediate",
    "description": "Display kernel routing policy database (RPDB) rules with ip rule.",
    "objective": "ip rule show",
    "hints": [
      "Command: ip rule show"
    ],
    "solution": "ip rule show",
    "accepted_regex": [
      "ip\\s+rule(\\s+show)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-69",
    "category": "Firewall & Network",
    "title": "Perform Forward DNS Lookup with Dig",
    "difficulty": "Easy",
    "description": "Query DNS A record for domain 'example.com' returning only concise answers (+short).",
    "objective": "dig +short example.com",
    "hints": [
      "Command: dig +short example.com"
    ],
    "solution": "dig +short example.com",
    "accepted_regex": [
      "dig\\s+\\+short\\s+example\\.com",
      "dig\\s+example\\.com\\s+\\+short"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-70",
    "category": "Firewall & Network",
    "title": "Perform Reverse DNS Pointer Lookup with Dig",
    "difficulty": "Easy",
    "description": "Perform reverse lookup for IP address 8.8.8.8 using dig -x.",
    "objective": "dig -x 8.8.8.8 +short",
    "hints": [
      "Command: dig -x 8.8.8.8 +short"
    ],
    "solution": "dig -x 8.8.8.8 +short",
    "accepted_regex": [
      "dig\\s+-x\\s+8\\.8\\.8\\.8(\\s+\\+short)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-71",
    "category": "Firewall & Network",
    "title": "Trace Network Path with traceroute / tracepath",
    "difficulty": "Easy",
    "description": "Trace MTU and hops to destination host 1.1.1.1 using tracepath.",
    "objective": "tracepath 1.1.1.1",
    "hints": [
      "Command: tracepath 1.1.1.1"
    ],
    "solution": "tracepath 1.1.1.1",
    "accepted_regex": [
      "(tracepath|traceroute)\\s+1\\.1\\.1\\.1"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-72",
    "category": "Firewall & Network",
    "title": "Download File with Curl Following HTTP Redirects",
    "difficulty": "Easy",
    "description": "Download remote URL http://example.com/file.tar.gz following 301/302 redirects (-L) saving as output (-O).",
    "objective": "curl -LO http://example.com/file.tar.gz",
    "hints": [
      "Command: curl -LO http://example.com/file.tar.gz"
    ],
    "solution": "curl -LO http://example.com/file.tar.gz",
    "accepted_regex": [
      "curl\\s+(-LO|-OL|-L\\s+-O|-O\\s+-L)\\s+http:\\/\\/example\\.com\\/file\\.tar\\.gz"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-73",
    "category": "Firewall & Network",
    "title": "Send HTTP POST JSON Payload with Curl",
    "difficulty": "Intermediate",
    "description": "Send POST request to http://localhost/api with JSON payload '{\"status\":\"ready\"}'.",
    "objective": "curl -X POST -H \"Content-Type: application/json\" -d '{\"status\":\"ready\"}' http://localhost/api",
    "hints": [
      "Command: curl -X POST -H \"Content-Type: application/json\" -d '{\"status\":\"ready\"}' http://localhost/api"
    ],
    "solution": "curl -X POST -H \"Content-Type: application/json\" -d '{\"status\":\"ready\"}' http://localhost/api",
    "accepted_regex": [
      "curl\\s+.*-X\\s+POST.*-H\\s+[\\\"']Content-Type:\\s+application\\/json[\\\"'].*http:\\/\\/localhost\\/api"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-74",
    "category": "Firewall & Network",
    "title": "Verify Chrony NTP Synchronization Sources",
    "difficulty": "Easy",
    "description": "Query chrony daemon for current time server sources and status.",
    "objective": "chronyc sources -v",
    "hints": [
      "Command: chronyc sources -v"
    ],
    "solution": "chronyc sources -v",
    "accepted_regex": [
      "chronyc\\s+sources(\\s+-v)?"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-75",
    "category": "Firewall & Network",
    "title": "Display NTP Tracking Offset and Jitter",
    "difficulty": "Easy",
    "description": "Inspect system clock offset, frequency, and jitter via chronyc tracking.",
    "objective": "chronyc tracking",
    "hints": [
      "Command: chronyc tracking"
    ],
    "solution": "chronyc tracking",
    "accepted_regex": [
      "chronyc\\s+tracking"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-76",
    "category": "Firewall & Network",
    "title": "Capture Network Packets on Interface with tcpdump",
    "difficulty": "Intermediate",
    "description": "Capture exactly 10 packets on interface ens192 without resolving DNS (-n).",
    "objective": "tcpdump -i ens192 -c 10 -n",
    "hints": [
      "Command: tcpdump -i ens192 -c 10 -n"
    ],
    "solution": "tcpdump -i ens192 -c 10 -n",
    "accepted_regex": [
      "tcpdump\\s+.*-i\\s+ens192.*(-c\\s+10.*-n|-n.*-c\\s+10)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-77",
    "category": "Firewall & Network",
    "title": "Filter TCP Port 80 Packets in tcpdump",
    "difficulty": "Intermediate",
    "description": "Listen on ens192 for TCP traffic specifically targeting port 80.",
    "objective": "tcpdump -i ens192 -n tcp port 80",
    "hints": [
      "Command: tcpdump -i ens192 -n tcp port 80"
    ],
    "solution": "tcpdump -i ens192 -n tcp port 80",
    "accepted_regex": [
      "tcpdump\\s+.*-i\\s+ens192.*tcp\\s+port\\s+80"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-78",
    "category": "Firewall & Network",
    "title": "Write Captured Packets to PCAP Capture File",
    "difficulty": "Intermediate",
    "description": "Capture 20 packets on ens192 and save to binary file /tmp/capture.pcap.",
    "objective": "tcpdump -i ens192 -c 20 -w /tmp/capture.pcap",
    "hints": [
      "Command: tcpdump -i ens192 -c 20 -w /tmp/capture.pcap"
    ],
    "solution": "tcpdump -i ens192 -c 20 -w /tmp/capture.pcap",
    "accepted_regex": [
      "tcpdump\\s+.*-i\\s+ens192.*-w\\s+\\/tmp\\/capture\\.pcap"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-79",
    "category": "Firewall & Network",
    "title": "Test Remote TCP Port Connectivity with nc / ncat",
    "difficulty": "Easy",
    "description": "Test if remote host 192.168.1.1 is listening on port 22 with 2-second timeout without sending data (-z).",
    "objective": "nc -zv -w 2 192.168.1.1 22",
    "hints": [
      "Command: nc -zv -w 2 192.168.1.1 22"
    ],
    "solution": "nc -zv -w 2 192.168.1.1 22",
    "accepted_regex": [
      "(nc|ncat)\\s+-(zv|vz)\\s+(-w\\s+2\\s+)?192\\.168\\.1\\.1\\s+22"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "net-80",
    "category": "Firewall & Network",
    "title": "Audit Network Interface MTU Size",
    "difficulty": "Easy",
    "description": "Change MTU size on device ens192 to jumbo frames 9000 using ip link.",
    "objective": "ip link set ens192 mtu 9000",
    "hints": [
      "Command: ip link set ens192 mtu 9000"
    ],
    "solution": "ip link set ens192 mtu 9000",
    "accepted_regex": [
      "ip\\s+link\\s+set\\s+ens192\\s+mtu\\s+9000"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-36",
    "category": "Grep & Regex",
    "title": "Match Only Digits of Specific Length",
    "difficulty": "Intermediate",
    "description": "Extract lines containing exactly 5 consecutive digits in codes.txt.",
    "objective": "grep -E '[0-9]{5}' codes.txt",
    "hints": [
      "Command: grep -E '[0-9]{5}' codes.txt"
    ],
    "solution": "grep -E '[0-9]{5}' codes.txt",
    "accepted_regex": [
      "grep\\s+(-E\\s+['\\\"]?\\[0-9\\]\\{5\\}['\\\"]?|'\\[0-9\\]\\{5\\}')\\s+codes\\.txt"
    ],
    "setup_files": {
      "codes.txt": "abc 12345 def\n1234\n999999\n54321\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-37",
    "category": "Grep & Regex",
    "title": "Extract Matching Substring Only with Flag -o",
    "difficulty": "Intermediate",
    "description": "Extract only the IPv4 addresses found inside system.log using grep -o -E.",
    "objective": "grep -o -E '[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' system.log",
    "hints": [
      "Command: grep -o -E '[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' system.log"
    ],
    "solution": "grep -o -E '[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}' system.log",
    "accepted_regex": [
      "grep\\s+(-oE|-Eo|-o\\s+-E|-E\\s+-o)\\s+.*system\\.log"
    ],
    "setup_files": {
      "system.log": "Connection from 192.168.1.50 port 22\nAuth failed for 10.0.0.99\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-38",
    "category": "Grep & Regex",
    "title": "Match Hexadecimal Color Codes",
    "difficulty": "Intermediate",
    "description": "Search styles.css for lines with 6-character hex color codes (#AABBCC).",
    "objective": "grep -E '#[0-9a-fA-F]{6}' styles.css",
    "hints": [
      "Command: grep -E '#[0-9a-fA-F]{6}' styles.css"
    ],
    "solution": "grep -E '#[0-9a-fA-F]{6}' styles.css",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]?#[0-9a-fA-F]\\{6\\}['\\\"]?\\s+styles\\.css"
    ],
    "setup_files": {
      "styles.css": "color: #ff0033;\nbackground: #123;\nborder: #FFFFFF;\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-39",
    "category": "Grep & Regex",
    "title": "Filter Lines Ending with Specific Word",
    "difficulty": "Easy",
    "description": "Find lines in test.txt that end strictly with the word 'FAILED'.",
    "objective": "grep 'FAILED$' test.txt",
    "hints": [
      "Command: grep 'FAILED$' test.txt"
    ],
    "solution": "grep 'FAILED$' test.txt",
    "accepted_regex": [
      "grep\\s+['\\\"]?FAILED\\$['\\\"]?\\s+test\\.txt"
    ],
    "setup_files": {
      "test.txt": "unit1 PASSED\nunit2 FAILED\nunit3 FAILED now\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-40",
    "category": "Grep & Regex",
    "title": "Find Lines Not Containing Any Numbers",
    "difficulty": "Intermediate",
    "description": "Display lines in roster.txt that contain zero digits.",
    "objective": "grep -v '[0-9]' roster.txt",
    "hints": [
      "Command: grep -v '[0-9]' roster.txt"
    ],
    "solution": "grep -v '[0-9]' roster.txt",
    "accepted_regex": [
      "grep\\s+-v\\s+['\\\"]?(\\[0-9\\]|\\[\\[:digit:\\]\\])['\\\"]?\\s+roster\\.txt"
    ],
    "setup_files": {
      "roster.txt": "Alice Smith\nBob 123\nCharlie Brown\nUser99\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-41",
    "category": "Grep & Regex",
    "title": "Match Repeated Characters Using Backreferences",
    "difficulty": "Advanced",
    "description": "Find words in dictionary.txt having doubled letters (like 'ee', 'oo', 'll') using backreferences.",
    "objective": "grep -E '([a-z])\\1' dictionary.txt",
    "hints": [
      "Command: grep -E '([a-z])\\1' dictionary.txt"
    ],
    "solution": "grep -E '([a-z])\\1' dictionary.txt",
    "accepted_regex": [
      "grep\\s+(-E\\s+['\\\"]?\\(\\[a-z\\]\\)\\\\\\1['\\\"]?|'([a-z])\\1')\\s+dictionary\\.txt"
    ],
    "setup_files": {
      "dictionary.txt": "book\ncat\napple\ndog\nfeed\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-42",
    "category": "Grep & Regex",
    "title": "Exclude Files Matching Pattern During Recursive Grep",
    "difficulty": "Intermediate",
    "description": "Search src/ recursively for 'main' excluding all *.log files.",
    "objective": "grep -r --exclude='*.log' 'main' src",
    "hints": [
      "Command: grep -r --exclude='*.log' 'main' src"
    ],
    "solution": "grep -r --exclude='*.log' 'main' src",
    "accepted_regex": [
      "grep\\s+.*-r.*--exclude=['\\\"]?\\*\\.log['\\\"].*src"
    ],
    "setup_files": {
      "src/app.py": "def main(): pass\n",
      "src/run.log": "main called\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-43",
    "category": "Grep & Regex",
    "title": "Exclude Directories During Recursive Grep",
    "difficulty": "Intermediate",
    "description": "Search . recursively for 'config' excluding the '.git' directory.",
    "objective": "grep -r --exclude-dir='.git' 'config' .",
    "hints": [
      "Command: grep -r --exclude-dir='.git' 'config' ."
    ],
    "solution": "grep -r --exclude-dir='.git' 'config' .",
    "accepted_regex": [
      "grep\\s+.*-r.*--exclude-dir=['\\\"]?\\.git['\\\"].*\\."
    ],
    "setup_files": {
      ".git/HEAD": "config ref\n",
      "app.conf": "config = 1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-44",
    "category": "Grep & Regex",
    "title": "Search for Pattern From Pattern File (-f)",
    "difficulty": "Intermediate",
    "description": "Search target.txt for keywords listed line-by-line in patterns.txt using grep -f.",
    "objective": "grep -f patterns.txt target.txt",
    "hints": [
      "Command: grep -f patterns.txt target.txt"
    ],
    "solution": "grep -f patterns.txt target.txt",
    "accepted_regex": [
      "grep\\s+-f\\s+patterns\\.txt\\s+target\\.txt"
    ],
    "setup_files": {
      "patterns.txt": "error\nwarning\n",
      "target.txt": "all ok\nfound error here\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-45",
    "category": "Grep & Regex",
    "title": "Suppress Normal Output with Quiet Mode Flag -q",
    "difficulty": "Easy",
    "description": "Test silently if 'prod_db' exists in hosts.txt without writing to stdout (grep -q).",
    "objective": "grep -q 'prod_db' hosts.txt",
    "hints": [
      "Command: grep -q 'prod_db' hosts.txt"
    ],
    "solution": "grep -q 'prod_db' hosts.txt",
    "accepted_regex": [
      "grep\\s+-q\\s+['\\\"]?prod_db['\\\"]?\\s+hosts\\.txt"
    ],
    "setup_files": {
      "hosts.txt": "10.0.0.1 prod_db\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-46",
    "category": "Grep & Regex",
    "title": "Search Multiple Files for Non-Matching Files (-L)",
    "difficulty": "Intermediate",
    "description": "List names of files among *.conf that do NOT contain the string 'ServerName'.",
    "objective": "grep -L 'ServerName' *.conf",
    "hints": [
      "Command: grep -L 'ServerName' *.conf"
    ],
    "solution": "grep -L 'ServerName' *.conf",
    "accepted_regex": [
      "grep\\s+-L\\s+['\\\"]?ServerName['\\\"]?\\s+\\*\\.conf"
    ],
    "setup_files": {
      "web1.conf": "ServerName site.com\n",
      "web2.conf": "Listen 80\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-47",
    "category": "Grep & Regex",
    "title": "Match Blank or Whitespace-Only Lines",
    "difficulty": "Easy",
    "description": "Count the number of blank or spaces-only lines in document.txt.",
    "objective": "grep -c '^[[:space:]]*$' document.txt",
    "hints": [
      "Command: grep -c '^[[:space:]]*$' document.txt"
    ],
    "solution": "grep -c '^[[:space:]]*$' document.txt",
    "accepted_regex": [
      "grep\\s+-c\\s+['\\\"]?\\^(\\[\\s\\]|\\[\\[:space:\\]\\]|\\s)\\*\\$['\\\"]?\\s+document\\.txt"
    ],
    "setup_files": {
      "document.txt": "line 1\n   \n\nline 2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-48",
    "category": "Grep & Regex",
    "title": "Match Optional Suffix with Extended Regex",
    "difficulty": "Easy",
    "description": "Match lines containing 'http' or 'https' using optional 's?' in protocol.txt.",
    "objective": "grep -E 'https?' protocol.txt",
    "hints": [
      "Command: grep -E 'https?' protocol.txt"
    ],
    "solution": "grep -E 'https?' protocol.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]?https\\?['\\\"]?\\s+protocol\\.txt"
    ],
    "setup_files": {
      "protocol.txt": "visit http://example\nvisit https://secure\nftp://other\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-49",
    "category": "Grep & Regex",
    "title": "Match Email Address Pattern with Grep -E",
    "difficulty": "Intermediate",
    "description": "Extract lines containing valid email address formats from contacts.txt.",
    "objective": "grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt",
    "hints": [
      "Command: grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt"
    ],
    "solution": "grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}' contacts.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+.*@.*contacts\\.txt"
    ],
    "setup_files": {
      "contacts.txt": "admin@example.com\nphone 555-1234\nuser.name@sub.domain.org\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-50",
    "category": "Grep & Regex",
    "title": "Match Lines With at Least 3 Words",
    "difficulty": "Intermediate",
    "description": "Find lines in sentences.txt having 3 or more whitespace-separated words.",
    "objective": "grep -E '^([^ ]+ +){2,}[^ ]+' sentences.txt",
    "hints": [
      "Command: grep -E '^([^ ]+ +){2,}[^ ]+' sentences.txt"
    ],
    "solution": "grep -E '^([^ ]+ +){2,}[^ ]+' sentences.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+.*sentences\\.txt"
    ],
    "setup_files": {
      "sentences.txt": "one\ntwo words\nthree words here\nfour words in line\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-51",
    "category": "Grep & Regex",
    "title": "Find Trailing Whitespace in Code Files",
    "difficulty": "Easy",
    "description": "Search script.sh for lines ending with unwanted trailing spaces or tabs.",
    "objective": "grep -n '[[:blank:]]$' script.sh",
    "hints": [
      "Command: grep -n '[[:blank:]]$' script.sh"
    ],
    "solution": "grep -n '[[:blank:]]$' script.sh",
    "accepted_regex": [
      "grep\\s+.*(\\[[:blank:]\\]|\\[ \\t\\]|\\\\s)\\$.*script\\.sh"
    ],
    "setup_files": {
      "script.sh": "echo clean\necho trailing   \nexit 0\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-52",
    "category": "Grep & Regex",
    "title": "Match Lines Starting with Uppercase Letter",
    "difficulty": "Easy",
    "description": "Extract lines from text.txt that start with a capital letter [A-Z].",
    "objective": "grep '^[A-Z]' text.txt",
    "hints": [
      "Command: grep '^[A-Z]' text.txt"
    ],
    "solution": "grep '^[A-Z]' text.txt",
    "accepted_regex": [
      "grep\\s+['\\\"]?\\^\\[A-Z\\]['\\\"]?\\s+text\\.txt"
    ],
    "setup_files": {
      "text.txt": "Apple is fruit\nbanana is yellow\nCarrot is root\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-53",
    "category": "Grep & Regex",
    "title": "Stop Reading File After N Matching Lines (-m)",
    "difficulty": "Easy",
    "description": "Search huge.log for 'CRITICAL' but stop searching after finding the first 3 matches.",
    "objective": "grep -m 3 'CRITICAL' huge.log",
    "hints": [
      "Command: grep -m 3 'CRITICAL' huge.log"
    ],
    "solution": "grep -m 3 'CRITICAL' huge.log",
    "accepted_regex": [
      "grep\\s+-m\\s*3\\s+['\\\"]?CRITICAL['\\\"]?\\s+huge\\.log"
    ],
    "setup_files": {
      "huge.log": "CRITICAL 1\nCRITICAL 2\nCRITICAL 3\nCRITICAL 4\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-54",
    "category": "Grep & Regex",
    "title": "Print Byte Offset of Matches with Flag -b",
    "difficulty": "Intermediate",
    "description": "Print the byte offset alongside matches for 'MAGIC' in binary.dat.",
    "objective": "grep -b 'MAGIC' binary.dat",
    "hints": [
      "Command: grep -b 'MAGIC' binary.dat"
    ],
    "solution": "grep -b 'MAGIC' binary.dat",
    "accepted_regex": [
      "grep\\s+-b\\s+['\\\"]?MAGIC['\\\"]?\\s+binary\\.dat"
    ],
    "setup_files": {
      "binary.dat": "header\nMAGIC_STRING\nfooter\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-55",
    "category": "Grep & Regex",
    "title": "Match MAC Hardware Addresses with ERE",
    "difficulty": "Intermediate",
    "description": "Find lines containing standard 6-byte colon-separated MAC addresses in arp.txt.",
    "objective": "grep -E '([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}' arp.txt",
    "hints": [
      "Command: grep -E '([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}' arp.txt"
    ],
    "solution": "grep -E '([0-9a-fA-F]{2}:){5}[0-9a-fA-F]{2}' arp.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+.*arp\\.txt"
    ],
    "setup_files": {
      "arp.txt": "host1 00:1A:2B:3C:4D:5E on ens192\nhost2 invalid:mac\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-56",
    "category": "Grep & Regex",
    "title": "Case-Insensitive Exact Word Match",
    "difficulty": "Easy",
    "description": "Find whole-word instances of 'admin' in any case (Admin, ADMIN) in users.txt.",
    "objective": "grep -iw 'admin' users.txt",
    "hints": [
      "Command: grep -iw 'admin' users.txt"
    ],
    "solution": "grep -iw 'admin' users.txt",
    "accepted_regex": [
      "grep\\s+(-iw|-wi|-i\\s+-w|-w\\s+-i)\\s+['\\\"]?admin['\\\"]?\\s+users\\.txt"
    ],
    "setup_files": {
      "users.txt": "Role: ADMIN\nadministrator\nRole: admin\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-57",
    "category": "Grep & Regex",
    "title": "Grep for Pattern Across Compressed Gz Logs with zgrep",
    "difficulty": "Intermediate",
    "description": "Search archived gzip log file /var/log/messages.1.gz for 'kernel panic' using zgrep.",
    "objective": "zgrep 'kernel panic' messages.1.gz",
    "hints": [
      "Command: zgrep 'kernel panic' messages.1.gz"
    ],
    "solution": "zgrep 'kernel panic' messages.1.gz",
    "accepted_regex": [
      "zgrep\\s+['\\\"]?kernel panic['\\\"]?\\s+messages\\.1\\.gz"
    ],
    "setup_files": {
      "messages.1.gz": "fake gz content with kernel panic\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-58",
    "category": "Grep & Regex",
    "title": "Match Lines Not Starting with Comment Character",
    "difficulty": "Easy",
    "description": "Print non-commented lines from /etc/fstab.test (lines that do not begin with '#').",
    "objective": "grep '^[^#]' fstab.test",
    "hints": [
      "Command: grep '^[^#]' fstab.test"
    ],
    "solution": "grep '^[^#]' fstab.test",
    "accepted_regex": [
      "grep\\s+['\\\"]?\\^\\[\\^#\\]['\\\"]?\\s+fstab\\.test"
    ],
    "setup_files": {
      "fstab.test": "# /etc/fstab\n/dev/sda1 / ext4 defaults 1 1\n# swap\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-59",
    "category": "Grep & Regex",
    "title": "Search for Non-ASCII Characters in Text",
    "difficulty": "Intermediate",
    "description": "Find lines containing non-ASCII unicode characters (byte values above 127) in data.txt.",
    "objective": "grep -P '[^\\x00-\\x7F]' data.txt",
    "hints": [
      "Command: grep -P '[^\\x00-\\x7F]' data.txt"
    ],
    "solution": "grep -P '[^\\x00-\\x7F]' data.txt",
    "accepted_regex": [
      "grep\\s+.*data\\.txt"
    ],
    "setup_files": {
      "data.txt": "plain ascii\naccent caf\u00e9\nmore plain\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-60",
    "category": "Grep & Regex",
    "title": "Match Port Numbers Between 1024 and 65535",
    "difficulty": "Advanced",
    "description": "Search ports.txt for lines containing high port numbers (1024-65535).",
    "objective": "grep -E '\\b(102[4-9]|10[3-9][0-9]|1[1-9][0-9]{2}|[2-5][0-9]{4}|6[0-4][0-9]{3}|65[0-4][0-9]{2}|655[0-2][0-9]|6553[0-5])\\b' ports.txt",
    "hints": [
      "Command: grep -E '\\b(102[4-9]|...)\\b' ports.txt"
    ],
    "solution": "grep -E '\\b(102[4-9]|10[3-9][0-9]|1[1-9][0-9]{2}|[2-5][0-9]{4}|6[0-4][0-9]{3}|65[0-4][0-9]{2}|655[0-2][0-9]|6553[0-5])\\b' ports.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+.*ports\\.txt"
    ],
    "setup_files": {
      "ports.txt": "port 80\nport 8080\nport 99999\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-61",
    "category": "Grep & Regex",
    "title": "Match Valid UUID Strings in Configs",
    "difficulty": "Intermediate",
    "description": "Search disk.conf for 8-4-4-4-12 standard UUID strings.",
    "objective": "grep -E '[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}' disk.conf",
    "hints": [
      "Command: grep -E '[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}' disk.conf"
    ],
    "solution": "grep -E '[0-9a-fA-F]{8}-([0-9a-fA-F]{4}-){3}[0-9a-fA-F]{12}' disk.conf",
    "accepted_regex": [
      "grep\\s+-E\\s+.*disk\\.conf"
    ],
    "setup_files": {
      "disk.conf": "UUID=c1b9d5a4-1234-5678-9abc-def012345678 /boot\nLABEL=root\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-62",
    "category": "Grep & Regex",
    "title": "Count Matches Per File Across Directory",
    "difficulty": "Easy",
    "description": "Count occurrences of 'error' in each individual *.log file.",
    "objective": "grep -c 'error' *.log",
    "hints": [
      "Command: grep -c 'error' *.log"
    ],
    "solution": "grep -c 'error' *.log",
    "accepted_regex": [
      "grep\\s+-c\\s+['\\\"]?error['\\\"]?\\s+\\*\\.log"
    ],
    "setup_files": {
      "a.log": "error\nerror\n",
      "b.log": "ok\nerror\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-63",
    "category": "Grep & Regex",
    "title": "Highlight Colored Matches in Interactive Terminal",
    "difficulty": "Easy",
    "description": "Search log.txt for 'root' forcing color output with --color=always.",
    "objective": "grep --color=always 'root' log.txt",
    "hints": [
      "Command: grep --color=always 'root' log.txt"
    ],
    "solution": "grep --color=always 'root' log.txt",
    "accepted_regex": [
      "grep\\s+--color(=always)?\\s+['\\\"]?root['\\\"]?\\s+log\\.txt"
    ],
    "setup_files": {
      "log.txt": "user root logged in\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-64",
    "category": "Grep & Regex",
    "title": "Search Multiple Exact Strings with fgrep / -F",
    "difficulty": "Easy",
    "description": "Search input.txt treating regex special characters ($*.) as raw literal characters (-F).",
    "objective": "grep -F '$*.' input.txt",
    "hints": [
      "Command: grep -F '$*.' input.txt"
    ],
    "solution": "grep -F '$*.' input.txt",
    "accepted_regex": [
      "(grep\\s+-F|fgrep)\\s+['\\\"]?\\\\\\$\\\\\\*\\.['\\\"]?\\s+input\\.txt",
      "grep\\s+-F\\s+['\\\"].*input\\.txt"
    ],
    "setup_files": {
      "input.txt": "symbol: $*.\nother: 123\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-65",
    "category": "Grep & Regex",
    "title": "Match Lines With Exactly One Word",
    "difficulty": "Intermediate",
    "description": "Find lines in words.txt consisting of exactly one word with no spaces.",
    "objective": "grep -E '^[a-zA-Z]+$' words.txt",
    "hints": [
      "Command: grep -E '^[a-zA-Z]+$' words.txt"
    ],
    "solution": "grep -E '^[a-zA-Z]+$' words.txt",
    "accepted_regex": [
      "grep\\s+(-E\\s+['\\\"]?\\^\\[a-zA-Z\\]\\+\\$['\\\"]?|'\\^[a-zA-Z]+\\$')\\s+words\\.txt"
    ],
    "setup_files": {
      "words.txt": "single\ntwo words\nanotherSingle\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-66",
    "category": "Grep & Regex",
    "title": "Match ISO 8601 Timestamp Patterns",
    "difficulty": "Intermediate",
    "description": "Extract lines from events.log matching YYYY-MM-DD format timestamps.",
    "objective": "grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log",
    "hints": [
      "Command: grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log"
    ],
    "solution": "grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}' events.log",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]?\\^\\[0-9\\]\\{4\\}-\\[0-9\\]\\{2\\}-\\[0-9\\]\\{2\\}['\\\"]?\\s+events\\.log"
    ],
    "setup_files": {
      "events.log": "2026-10-09 Event started\nInvalid date event\n2026-12-31 Event closed\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-67",
    "category": "Grep & Regex",
    "title": "Display Lines Before Match (-B) Only",
    "difficulty": "Easy",
    "description": "Show 3 lines of leading context before every occurrence of 'PANIC' in trace.txt.",
    "objective": "grep -B 3 'PANIC' trace.txt",
    "hints": [
      "Command: grep -B 3 'PANIC' trace.txt"
    ],
    "solution": "grep -B 3 'PANIC' trace.txt",
    "accepted_regex": [
      "grep\\s+-B\\s*3\\s+['\\\"]?PANIC['\\\"]?\\s+trace\\.txt"
    ],
    "setup_files": {
      "trace.txt": "step 1\nstep 2\nstep 3\nPANIC: error\nstep 4\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-68",
    "category": "Grep & Regex",
    "title": "Display Lines After Match (-A) Only",
    "difficulty": "Easy",
    "description": "Show 2 lines of trailing context following 'START' in job.log.",
    "objective": "grep -A 2 'START' job.log",
    "hints": [
      "Command: grep -A 2 'START' job.log"
    ],
    "solution": "grep -A 2 'START' job.log",
    "accepted_regex": [
      "grep\\s+-A\\s*2\\s+['\\\"]?START['\\\"]?\\s+job\\.log"
    ],
    "setup_files": {
      "job.log": "init\nSTART\nrun task 1\nrun task 2\nfinish\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-69",
    "category": "Grep & Regex",
    "title": "Match Lines With Only Digits and Commas",
    "difficulty": "Intermediate",
    "description": "Search csv.txt for lines containing exclusively numeric digits and commas.",
    "objective": "grep -E '^[0-9,]+$' csv.txt",
    "hints": [
      "Command: grep -E '^[0-9,]+$' csv.txt"
    ],
    "solution": "grep -E '^[0-9,]+$' csv.txt",
    "accepted_regex": [
      "grep\\s+-E\\s+['\\\"]?\\^\\[0-9,\\]\\+\\$['\\\"]?\\s+csv\\.txt"
    ],
    "setup_files": {
      "csv.txt": "10,20,30\nabc,12,34\n100,200\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "grep-70",
    "category": "Grep & Regex",
    "title": "Search for Empty Strings in Quoted Attributes",
    "difficulty": "Intermediate",
    "description": "Find lines in settings.env having empty quoted variables like VAR=\"\".",
    "objective": "grep '=\"\"' settings.env",
    "hints": [
      "Command: grep '=\"\"' settings.env"
    ],
    "solution": "grep '=\"\"' settings.env",
    "accepted_regex": [
      "grep\\s+['\\\"]?=\\\"\\\"['\\\"]?\\s+settings\\.env"
    ],
    "setup_files": {
      "settings.env": "HOST=\"localhost\"\nTOKEN=\"\"\nPORT=\"3000\"\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-31",
    "category": "Sed Stream Editing",
    "title": "Insert Text Before Matched Pattern",
    "difficulty": "Intermediate",
    "description": "Insert line '# Begin Config' immediately before the line matching 'ServerRoot' in httpd.conf.",
    "objective": "sed '/ServerRoot/i # Begin Config' httpd.conf",
    "hints": [
      "Command: sed '/ServerRoot/i # Begin Config' httpd.conf"
    ],
    "solution": "sed '/ServerRoot/i # Begin Config' httpd.conf",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/ServerRoot\\/i\\s+# Begin Config['\\\"]\\s+httpd\\.conf"
    ],
    "setup_files": {
      "httpd.conf": "Listen 80\nServerRoot /etc/httpd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-32",
    "category": "Sed Stream Editing",
    "title": "Replace Entire Line Containing Pattern (c Command)",
    "difficulty": "Intermediate",
    "description": "Change the line matching 'SELINUX=' to 'SELINUX=enforcing' using sed c command.",
    "objective": "sed '/^SELINUX=/c SELINUX=enforcing' config",
    "hints": [
      "Command: sed '/^SELINUX=/c SELINUX=enforcing' config"
    ],
    "solution": "sed '/^SELINUX=/c SELINUX=enforcing' config",
    "accepted_regex": [
      "sed\\s+['\\\"](\\/\\^SELINUX=\\/c\\s+SELINUX=enforcing|\\/\\^SELINUX=\\/c\\\\SELINUX=enforcing)['\\\"]\\s+config"
    ],
    "setup_files": {
      "config": "# selinux\nSELINUX=disabled\nSELINUXTYPE=targeted\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-33",
    "category": "Sed Stream Editing",
    "title": "Print Only Even Numbered Lines with Sed",
    "difficulty": "Intermediate",
    "description": "Print only the even-numbered lines (2, 4, 6, ...) of numbers.txt using sed address step notation.",
    "objective": "sed -n '2~2p' numbers.txt",
    "hints": [
      "Command: sed -n '2~2p' numbers.txt"
    ],
    "solution": "sed -n '2~2p' numbers.txt",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]2~2p['\\\"]\\s+numbers\\.txt"
    ],
    "setup_files": {
      "numbers.txt": "one\ntwo\nthree\nfour\nfive\nsix\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-34",
    "category": "Sed Stream Editing",
    "title": "Print Only Odd Numbered Lines with Sed",
    "difficulty": "Intermediate",
    "description": "Print only odd-numbered lines (1, 3, 5, ...) of numbers.txt with sed.",
    "objective": "sed -n '1~2p' numbers.txt",
    "hints": [
      "Command: sed -n '1~2p' numbers.txt"
    ],
    "solution": "sed -n '1~2p' numbers.txt",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]1~2p['\\\"]\\s+numbers\\.txt"
    ],
    "setup_files": {
      "numbers.txt": "one\ntwo\nthree\nfour\nfive\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-35",
    "category": "Sed Stream Editing",
    "title": "Remove Windows Carriage Return Line Endings",
    "difficulty": "Easy",
    "description": "Strip DOS/Windows carriage return (\\r) characters from dos.txt using sed.",
    "objective": "sed 's/\\r$//' dos.txt",
    "hints": [
      "Command: sed 's/\\r$//' dos.txt"
    ],
    "solution": "sed 's/\\r$//' dos.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\\\r\\$?\\/\\/['\\\"]\\s+dos\\.txt"
    ],
    "setup_files": {
      "dos.txt": "line1\r\nline2\r\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-36",
    "category": "Sed Stream Editing",
    "title": "Replace Only First Occurrence per Line",
    "difficulty": "Easy",
    "description": "Replace only the first instance of 'cat' with 'dog' per line in pets.txt (omit /g).",
    "objective": "sed 's/cat/dog/' pets.txt",
    "hints": [
      "Command: sed 's/cat/dog/' pets.txt"
    ],
    "solution": "sed 's/cat/dog/' pets.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/cat\\/dog\\/['\\\"]\\s+pets\\.txt"
    ],
    "setup_files": {
      "pets.txt": "cat and cat\nmy cat has a cat\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-37",
    "category": "Sed Stream Editing",
    "title": "Replace Only Second Occurrence on Each Line",
    "difficulty": "Intermediate",
    "description": "Replace specifically the 2nd instance of 'foo' with 'bar' on each line using flag '2'.",
    "objective": "sed 's/foo/bar/2' text.txt",
    "hints": [
      "Command: sed 's/foo/bar/2' text.txt"
    ],
    "solution": "sed 's/foo/bar/2' text.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/foo\\/bar\\/2['\\\"]\\s+text\\.txt"
    ],
    "setup_files": {
      "text.txt": "foo foo foo\nfoo foo\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-38",
    "category": "Sed Stream Editing",
    "title": "In-Place Backup Creation with sed -i.bak",
    "difficulty": "Intermediate",
    "description": "Perform in-place substitution changing 'v1' to 'v2' in app.conf creating backup file app.conf.bak.",
    "objective": "sed -i.bak 's/v1/v2/g' app.conf",
    "hints": [
      "Command: sed -i.bak 's/v1/v2/g' app.conf"
    ],
    "solution": "sed -i.bak 's/v1/v2/g' app.conf",
    "accepted_regex": [
      "sed\\s+-i\\.bak\\s+['\\\"]s\\/v1\\/v2\\/g['\\\"]\\s+app\\.conf"
    ],
    "setup_files": {
      "app.conf": "version = v1\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-39",
    "category": "Sed Stream Editing",
    "title": "Delete Trailing Spaces from End of Every Line",
    "difficulty": "Easy",
    "description": "Remove all whitespace characters from line ends in messy.txt.",
    "objective": "sed 's/[[:blank:]]*$//' messy.txt",
    "hints": [
      "Command: sed 's/[[:blank:]]*$//' messy.txt"
    ],
    "solution": "sed 's/[[:blank:]]*$//' messy.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/(\\[[:blank:]\\]\\*|\\[ \\\\t\\]\\*|\\\\s\\*)\\$\\/\\/['\\\"]\\s+messy\\.txt"
    ],
    "setup_files": {
      "messy.txt": "hello   \nworld\t\t\nclean\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-40",
    "category": "Sed Stream Editing",
    "title": "Double Space a File (Add Blank Line After Each Line)",
    "difficulty": "Intermediate",
    "description": "Add an empty blank line after each line in notes.txt using the 'G' command.",
    "objective": "sed 'G' notes.txt",
    "hints": [
      "Command: sed 'G' notes.txt"
    ],
    "solution": "sed 'G' notes.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]G['\\\"]\\s+notes\\.txt"
    ],
    "setup_files": {
      "notes.txt": "line1\nline2\nline3\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-41",
    "category": "Sed Stream Editing",
    "title": "Print Only Lines Matching Pattern (Mimic Grep)",
    "difficulty": "Easy",
    "description": "Use sed -n with the 'p' flag to print only lines containing 'CRITICAL' from err.log.",
    "objective": "sed -n '/CRITICAL/p' err.log",
    "hints": [
      "Command: sed -n '/CRITICAL/p' err.log"
    ],
    "solution": "sed -n '/CRITICAL/p' err.log",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]\\/CRITICAL\\/p['\\\"]\\s+err\\.log"
    ],
    "setup_files": {
      "err.log": "INFO: ok\nCRITICAL: crash\nDEBUG: trace\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-42",
    "category": "Sed Stream Editing",
    "title": "Quit Sed Processing Upon First Match (q Command)",
    "difficulty": "Intermediate",
    "description": "Print lines of stream.txt until reaching the line with 'STOP' and immediately exit.",
    "objective": "sed '/STOP/q' stream.txt",
    "hints": [
      "Command: sed '/STOP/q' stream.txt"
    ],
    "solution": "sed '/STOP/q' stream.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/STOP\\/q['\\\"]\\s+stream\\.txt"
    ],
    "setup_files": {
      "stream.txt": "part1\npart2\nSTOP\npart3\npart4\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-43",
    "category": "Sed Stream Editing",
    "title": "Uppercase Matched Words Using \\U Substitution",
    "difficulty": "Advanced",
    "description": "Convert all instances of 'warning' to uppercase 'WARNING' using \\U in sed.",
    "objective": "sed 's/warning/\\U&/g' log.txt",
    "hints": [
      "Command: sed 's/warning/\\U&/g' log.txt"
    ],
    "solution": "sed 's/warning/\\U&/g' log.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/warning\\/\\\\U&\\/g['\\\"]\\s+log\\.txt"
    ],
    "setup_files": {
      "log.txt": "this is a warning: check disk warning\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-44",
    "category": "Sed Stream Editing",
    "title": "Lowercase Matched Words Using \\L Substitution",
    "difficulty": "Advanced",
    "description": "Convert uppercase 'ALERT' to lowercase 'alert' using \\L in sed.",
    "objective": "sed 's/ALERT/\\L&/g' log.txt",
    "hints": [
      "Command: sed 's/ALERT/\\L&/g' log.txt"
    ],
    "solution": "sed 's/ALERT/\\L&/g' log.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/ALERT\\/\\\\L&\\/g['\\\"]\\s+log\\.txt"
    ],
    "setup_files": {
      "log.txt": "SECURITY ALERT DETECTED\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-45",
    "category": "Sed Stream Editing",
    "title": "Replace Pattern Only Within Line Number Range",
    "difficulty": "Intermediate",
    "description": "Replace 'temp' with 'perm' only between lines 10 and 20 in data.txt.",
    "objective": "sed '10,20s/temp/perm/g' data.txt",
    "hints": [
      "Command: sed '10,20s/temp/perm/g' data.txt"
    ],
    "solution": "sed '10,20s/temp/perm/g' data.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]10,20s\\/temp\\/perm\\/g['\\\"]\\s+data\\.txt"
    ],
    "setup_files": {
      "data.txt": "line 1 temp\nline 2 temp\nline 3 temp\nline 4 temp\nline 5 temp\nline 6 temp\nline 7 temp\nline 8 temp\nline 9 temp\nline 10 temp\nline 11 temp\nline 12 temp\nline 13 temp\nline 14 temp\nline 15 temp\nline 16 temp\nline 17 temp\nline 18 temp\nline 19 temp\nline 20 temp\nline 21 temp\nline 22 temp\nline 23 temp\nline 24 temp\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-46",
    "category": "Sed Stream Editing",
    "title": "Read External File Contents into Stream (r Command)",
    "difficulty": "Intermediate",
    "description": "Insert the content of header.txt right after the line matching '<body>' in page.html.",
    "objective": "sed '/<body>/r header.txt' page.html",
    "hints": [
      "Command: sed '/<body>/r header.txt' page.html"
    ],
    "solution": "sed '/<body>/r header.txt' page.html",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/<body>\\/r\\s+header\\.txt['\\\"]\\s+page\\.html"
    ],
    "setup_files": {
      "page.html": "<html>\n<body>\n<p>content</p>\n</body>\n</html>\n",
      "header.txt": "<h1>Header</h1>\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-47",
    "category": "Sed Stream Editing",
    "title": "Write Matched Lines to Separate File (w Command)",
    "difficulty": "Intermediate",
    "description": "Write all lines containing 'ERROR' to new file errors.log using sed w command.",
    "objective": "sed -n '/ERROR/w errors.log' app.log",
    "hints": [
      "Command: sed -n '/ERROR/w errors.log' app.log"
    ],
    "solution": "sed -n '/ERROR/w errors.log' app.log",
    "accepted_regex": [
      "sed\\s+(-n\\s+)?['\\\"]\\/ERROR\\/w\\s+errors\\.log['\\\"]\\s+app\\.log"
    ],
    "setup_files": {
      "app.log": "INFO: ok\nERROR: crash\nWARN: low\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-48",
    "category": "Sed Stream Editing",
    "title": "Swap Two Adjacent Words on Each Line",
    "difficulty": "Advanced",
    "description": "Swap the first two words on each line in pairs.txt using capture groups (\\1 and \\2).",
    "objective": "sed -E 's/^([a-zA-Z]+) ([a-zA-Z]+)/\\2 \\1/' pairs.txt",
    "hints": [
      "Command: sed -E 's/^([a-zA-Z]+) ([a-zA-Z]+)/\\2 \\1/' pairs.txt"
    ],
    "solution": "sed -E 's/^([a-zA-Z]+) ([a-zA-Z]+)/\\2 \\1/' pairs.txt",
    "accepted_regex": [
      "sed\\s+-E\\s+['\\\"]s\\/\\^\\(\\[a-zA-Z\\]\\+\\)\\s+\\(\\[a-zA-Z\\]\\+\\)\\/\\\\2\\s+\\\\1\\/['\\\"]\\s+pairs\\.txt"
    ],
    "setup_files": {
      "pairs.txt": "first second\nhello world\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-49",
    "category": "Sed Stream Editing",
    "title": "Strip HTML Tags from Document",
    "difficulty": "Intermediate",
    "description": "Remove all <...> tags from markup.html using sed substitution.",
    "objective": "sed 's/<[^>]*>//g' markup.html",
    "hints": [
      "Command: sed 's/<[^>]*>//g' markup.html"
    ],
    "solution": "sed 's/<[^>]*>//g' markup.html",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/<\\[\\^>\\]\\*>\\/\\/g['\\\"]\\s+markup\\.html"
    ],
    "setup_files": {
      "markup.html": "<h1>Title</h1>\n<p>This is <b>bold</b> text.</p>\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-50",
    "category": "Sed Stream Editing",
    "title": "Print Total Line Count with Sed = Command",
    "difficulty": "Intermediate",
    "description": "Print the total line count of book.txt by executing the '=' command on the last line ($).",
    "objective": "sed -n '$=' book.txt",
    "hints": [
      "Command: sed -n '$=' book.txt"
    ],
    "solution": "sed -n '$=' book.txt",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]\\\\?\\$=['\\\"]\\s+book\\.txt"
    ],
    "setup_files": {
      "book.txt": "chapter 1\nchapter 2\nchapter 3\nchapter 4\nchapter 5\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-51",
    "category": "Sed Stream Editing",
    "title": "Indent Every Line by 4 Spaces",
    "difficulty": "Easy",
    "description": "Prepend 4 space characters to the start of each line in code.txt.",
    "objective": "sed 's/^/    /' code.txt",
    "hints": [
      "Command: sed 's/^/    /' code.txt"
    ],
    "solution": "sed 's/^/    /' code.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\^\\/    \\/['\\\"]\\s+code\\.txt"
    ],
    "setup_files": {
      "code.txt": "def run():\nreturn True\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-52",
    "category": "Sed Stream Editing",
    "title": "Remove Consecutive Blank Lines",
    "difficulty": "Advanced",
    "description": "Compress multiple consecutive blank lines into a single blank line in spaces.txt.",
    "objective": "sed '/^$/N;/\\n$/D' spaces.txt",
    "hints": [
      "Command: sed '/^$/N;/\\n$/D' spaces.txt"
    ],
    "solution": "sed '/^$/N;/\\n$/D' spaces.txt",
    "accepted_regex": [
      "sed\\s+.*spaces\\.txt"
    ],
    "setup_files": {
      "spaces.txt": "a\n\n\n\nb\n\nc\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-53",
    "category": "Sed Stream Editing",
    "title": "Mask Sensitive Credit Card Numbers",
    "difficulty": "Advanced",
    "description": "Mask 16-digit credit card numbers in cards.txt to 'XXXX-XXXX-XXXX-1234'.",
    "objective": "sed -E 's/[0-9]{4}-[0-9]{4}-[0-9]{4}-([0-9]{4})/XXXX-XXXX-XXXX-\\1/g' cards.txt",
    "hints": [
      "Command: sed -E 's/[0-9]{4}-[0-9]{4}-[0-9]{4}-([0-9]{4})/XXXX-XXXX-XXXX-\\1/g' cards.txt"
    ],
    "solution": "sed -E 's/[0-9]{4}-[0-9]{4}-[0-9]{4}-([0-9]{4})/XXXX-XXXX-XXXX-\\1/g' cards.txt",
    "accepted_regex": [
      "sed\\s+-E\\s+.*cards\\.txt"
    ],
    "setup_files": {
      "cards.txt": "user paid with 1234-5678-9012-3456\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-54",
    "category": "Sed Stream Editing",
    "title": "Print Line Number Along with Matched Line",
    "difficulty": "Intermediate",
    "description": "Print the line number before any line matching 'FATAL' in system.log using {=; p}.",
    "objective": "sed -n '/FATAL/{=; p}' system.log",
    "hints": [
      "Command: sed -n '/FATAL/{=; p}' system.log"
    ],
    "solution": "sed -n '/FATAL/{=; p}' system.log",
    "accepted_regex": [
      "sed\\s+-n\\s+['\\\"]\\/FATAL\\/\\{=\\s*;\\s*p\\}['\\\"]\\s+system\\.log"
    ],
    "setup_files": {
      "system.log": "ok\nok\nFATAL: error\nok\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-55",
    "category": "Sed Stream Editing",
    "title": "Delete All Lines After Pattern Until End of File",
    "difficulty": "Intermediate",
    "description": "Delete from the line containing '---END---' all the way through the end of file ($) in message.txt.",
    "objective": "sed '/---END---/,$d' message.txt",
    "hints": [
      "Command: sed '/---END---/,$d' message.txt"
    ],
    "solution": "sed '/---END---/,$d' message.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]\\/---END---\\/,\\$d['\\\"]\\s+message\\.txt"
    ],
    "setup_files": {
      "message.txt": "header\nbody\n---END---\nfooter\nextra\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-56",
    "category": "Sed Stream Editing",
    "title": "Remove Semicolon Comments from INI File",
    "difficulty": "Easy",
    "description": "Delete lines starting with a semicolon ';' comment from php.ini.",
    "objective": "sed '/^[[:blank:]]*;/d' php.ini",
    "hints": [
      "Command: sed '/^[[:blank:]]*;/d' php.ini"
    ],
    "solution": "sed '/^[[:blank:]]*;/d' php.ini",
    "accepted_regex": [
      "sed\\s+['\\\"](\\/\\^\\[\\[:blank:\\]\\]\\*;\\/d|\\/\\^;\\/d)['\\\"]\\s+php\\.ini"
    ],
    "setup_files": {
      "php.ini": "; php settings\nmemory_limit = 128M\n; max time\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-57",
    "category": "Sed Stream Editing",
    "title": "Join Lines Ending with Backslash",
    "difficulty": "Advanced",
    "description": "Join lines ending with a trailing backslash continuation character in script.sh.",
    "objective": "sed -e :a -e '/\\\\$/N; s/\\\\\\n//; ta' script.sh",
    "hints": [
      "Command: sed -e :a -e '/\\\\$/N; s/\\\\\\n//; ta' script.sh"
    ],
    "solution": "sed -e :a -e '/\\\\$/N; s/\\\\\\n//; ta' script.sh",
    "accepted_regex": [
      "sed\\s+.*script\\.sh"
    ],
    "setup_files": {
      "script.sh": "echo hello \\\nworld\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-58",
    "category": "Sed Stream Editing",
    "title": "Extract Text Contained Inside Parentheses",
    "difficulty": "Intermediate",
    "description": "Extract only the contents inside parentheses '(foo)' on each line of data.txt.",
    "objective": "sed -n 's/.*(\\([^)]*\\)).*/\\1/p' data.txt",
    "hints": [
      "Command: sed -n 's/.*(\\([^)]*\\)).*/\\1/p' data.txt"
    ],
    "solution": "sed -n 's/.*(\\([^)]*\\)).*/\\1/p' data.txt",
    "accepted_regex": [
      "sed\\s+-n\\s+.*data\\.txt"
    ],
    "setup_files": {
      "data.txt": "name (John Doe) age\nserver (prod-01) online\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-59",
    "category": "Sed Stream Editing",
    "title": "Change Slashes in URL Without Escaping Using Hash Delimiter",
    "difficulty": "Easy",
    "description": "Replace 'http://old.com/api' with 'https://new.com/v2' using '#' delimiter in sed.",
    "objective": "sed 's#http://old.com/api#https://new.com/v2#g' urls.txt",
    "hints": [
      "Command: sed 's#http://old.com/api#https://new.com/v2#g' urls.txt"
    ],
    "solution": "sed 's#http://old.com/api#https://new.com/v2#g' urls.txt",
    "accepted_regex": [
      "sed\\s+['\\\"]s#http:\\/\\/old\\.com\\/api#https:\\/\\/new\\.com\\/v2#g['\\\"]\\s+urls\\.txt"
    ],
    "setup_files": {
      "urls.txt": "endpoint = http://old.com/api/users\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "sed-60",
    "category": "Sed Stream Editing",
    "title": "Delete Everything Before First Occurrence of Delimiter",
    "difficulty": "Intermediate",
    "description": "Strip everything up to and including the first colon ':' on each line of /etc/passwd sample.",
    "objective": "sed 's/^[^:]*://' passwd.sample",
    "hints": [
      "Command: sed 's/^[^:]*://' passwd.sample"
    ],
    "solution": "sed 's/^[^:]*://' passwd.sample",
    "accepted_regex": [
      "sed\\s+['\\\"]s\\/\\^\\[\\^:\\]\\*:\\/\\/['\\\"]\\s+passwd\\.sample"
    ],
    "setup_files": {
      "passwd.sample": "root:x:0:0:root:/root:/bin/bash\nbin:x:1:1:bin:/bin:/sbin/nologin\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-31",
    "category": "Awk Processing",
    "title": "Compute Sum and Average with AWK",
    "difficulty": "Intermediate",
    "description": "Calculate both the sum and average of column 2 values in stats.tsv.",
    "objective": "awk '{sum+=$2} END {print \"Sum:\", sum, \"Avg:\", sum/NR}' stats.tsv",
    "hints": [
      "Command: awk '{sum+=$2} END {print \"Sum:\", sum, \"Avg:\", sum/NR}' stats.tsv"
    ],
    "solution": "awk '{sum+=$2} END {print \"Sum:\", sum, \"Avg:\", sum/NR}' stats.tsv",
    "accepted_regex": [
      "awk\\s+.*sum\\+=\\$2.*stats\\.tsv"
    ],
    "setup_files": {
      "stats.tsv": "item1\t10\nitem2\t20\nitem3\t30\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-32",
    "category": "Awk Processing",
    "title": "Format Column Outputs with printf Table Padding",
    "difficulty": "Intermediate",
    "description": "Format columns 1 (left-aligned 15 chars) and 3 (right-aligned 8 chars) in table.txt.",
    "objective": "awk '{printf \"%-15s %8s\\n\", $1, $3}' table.txt",
    "hints": [
      "Command: awk '{printf \"%-15s %8s\\n\", $1, $3}' table.txt"
    ],
    "solution": "awk '{printf \"%-15s %8s\\n\", $1, $3}' table.txt",
    "accepted_regex": [
      "awk\\s+.*printf.*table\\.txt"
    ],
    "setup_files": {
      "table.txt": "alpha 1 100\nbeta 2 2500\ngamma 3 9\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-33",
    "category": "Awk Processing",
    "title": "Find Maximum Value in Column with Tracking Record",
    "difficulty": "Intermediate",
    "description": "Find the highest numeric score in column 2 and print the winner's name ($1) and score ($2).",
    "objective": "awk '$2 > max {max=$2; winner=$1} END {print winner, max}' scores.txt",
    "hints": [
      "Command: awk '$2 > max {max=$2; winner=$1} END {print winner, max}' scores.txt"
    ],
    "solution": "awk '$2 > max {max=$2; winner=$1} END {print winner, max}' scores.txt",
    "accepted_regex": [
      "awk\\s+.*max.*scores\\.txt"
    ],
    "setup_files": {
      "scores.txt": "Alice 88\nBob 95\nCharlie 91\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-34",
    "category": "Awk Processing",
    "title": "Filter Lines by String Length Function",
    "difficulty": "Easy",
    "description": "Print lines from phrases.txt where length of line exceeds 40 characters using length().",
    "objective": "awk 'length($0) > 40' phrases.txt",
    "hints": [
      "Command: awk 'length($0) > 40' phrases.txt"
    ],
    "solution": "awk 'length($0) > 40' phrases.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]length(\\(\\$0\\)?|\\$0)?\\s*>\\s*40['\\\"]\\s+phrases\\.txt"
    ],
    "setup_files": {
      "phrases.txt": "short\nthis is a longer sentence exceeding forty characters easily here\nanother medium phrase\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-35",
    "category": "Awk Processing",
    "title": "Count Occurrences of Values Using Associative Arrays",
    "difficulty": "Intermediate",
    "description": "Count how many times each HTTP method ($1) appears in access.log.",
    "objective": "awk '{count[$1]++} END {for (m in count) print m, count[m]}' access.log",
    "hints": [
      "Command: awk '{count[$1]++} END {for (m in count) print m, count[m]}' access.log"
    ],
    "solution": "awk '{count[$1]++} END {for (m in count) print m, count[m]}' access.log",
    "accepted_regex": [
      "awk\\s+.*count\\[\\$1\\]\\+\\+.*access\\.log"
    ],
    "setup_files": {
      "access.log": "GET /index\nPOST /login\nGET /style.css\nGET /logo.png\nPOST /api\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-36",
    "category": "Awk Processing",
    "title": "Substituted Text in Specific Field with gsub()",
    "difficulty": "Intermediate",
    "description": "Replace all slashes '/' with dashes '-' only in field 2 of paths.txt.",
    "objective": "awk '{gsub(\"/\", \"-\", $2); print}' paths.txt",
    "hints": [
      "Command: awk '{gsub(\"/\", \"-\", $2); print}' paths.txt"
    ],
    "solution": "awk '{gsub(\"/\", \"-\", $2); print}' paths.txt",
    "accepted_regex": [
      "awk\\s+.*gsub.*paths\\.txt"
    ],
    "setup_files": {
      "paths.txt": "server1 /var/log/app active\nserver2 /etc/httpd standby\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-37",
    "category": "Awk Processing",
    "title": "Print Fields in Reverse Order",
    "difficulty": "Intermediate",
    "description": "Print all fields of each line in reverse order from last (NF) down to 1.",
    "objective": "awk '{for(i=NF; i>0; i--) printf \"%s \", $i; print \"\"}' words.txt",
    "hints": [
      "Command: awk '{for(i=NF; i>0; i--) printf \"%s \", $i; print \"\"}' words.txt"
    ],
    "solution": "awk '{for(i=NF; i>0; i--) printf \"%s \", $i; print \"\"}' words.txt",
    "accepted_regex": [
      "awk\\s+.*for.*NF.*words\\.txt"
    ],
    "setup_files": {
      "words.txt": "one two three four\nalpha beta gamma\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-38",
    "category": "Awk Processing",
    "title": "Extract Field Matching Regex with match()",
    "difficulty": "Advanced",
    "description": "Extract the IP address from log message in syslog.txt using match() and substr().",
    "objective": "awk 'match($0, /[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+/) {print substr($0, RSTART, RLENGTH)}' syslog.txt",
    "hints": [
      "Command: awk 'match($0, /[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+/) {print substr($0, RSTART, RLENGTH)}' syslog.txt"
    ],
    "solution": "awk 'match($0, /[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+/) {print substr($0, RSTART, RLENGTH)}' syslog.txt",
    "accepted_regex": [
      "awk\\s+.*match.*substr.*syslog\\.txt"
    ],
    "setup_files": {
      "syslog.txt": "User connected from 192.168.1.100 port 5021\nDropped from 10.0.0.5\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-39",
    "category": "Awk Processing",
    "title": "Multi-Character Field Separator",
    "difficulty": "Intermediate",
    "description": "Parse log.txt separated by '::' and print field 1 and field 3.",
    "objective": "awk -F '::' '{print $1, $3}' log.txt",
    "hints": [
      "Command: awk -F '::' '{print $1, $3}' log.txt"
    ],
    "solution": "awk -F '::' '{print $1, $3}' log.txt",
    "accepted_regex": [
      "awk\\s+-F\\s+['\\\"]::['\\\"]\\s+.*log\\.txt"
    ],
    "setup_files": {
      "log.txt": "2026-10-09::auth::SUCCESS::user1\n2026-10-09::ssh::FAILED::root\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-40",
    "category": "Awk Processing",
    "title": "Custom Output Field Separator (OFS)",
    "difficulty": "Easy",
    "description": "Read whitespace-delimited columns and output them separated by commas (OFS=\",\").",
    "objective": "awk -v OFS=',' '{print $1, $2, $3}' data.txt",
    "hints": [
      "Command: awk -v OFS=',' '{print $1, $2, $3}' data.txt"
    ],
    "solution": "awk -v OFS=',' '{print $1, $2, $3}' data.txt",
    "accepted_regex": [
      "awk\\s+.*OFS=.*data\\.txt"
    ],
    "setup_files": {
      "data.txt": "a b c\nd e f\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-41",
    "category": "Awk Processing",
    "title": "Process Multi-Line Records Separated by Blank Lines",
    "difficulty": "Advanced",
    "description": "Set record separator RS='' (paragraph mode) to process multi-line stanzas and print the first line of each record.",
    "objective": "awk 'BEGIN {RS=\"\"} {print $1}' stanzas.txt",
    "hints": [
      "Command: awk 'BEGIN {RS=\"\"} {print $1}' stanzas.txt"
    ],
    "solution": "awk 'BEGIN {RS=\"\"} {print $1}' stanzas.txt",
    "accepted_regex": [
      "awk\\s+.*RS=.*stanzas\\.txt"
    ],
    "setup_files": {
      "stanzas.txt": "Record1 Line1\nRecord1 Line2\n\nRecord2 Line1\nRecord2 Line2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-42",
    "category": "Awk Processing",
    "title": "Compute Running Total on Column",
    "difficulty": "Intermediate",
    "description": "Print each line along with the cumulative running total of field 2.",
    "objective": "awk '{total += $2; print $0, \"Total:\" total}' ledger.txt",
    "hints": [
      "Command: awk '{total += $2; print $0, \"Total:\" total}' ledger.txt"
    ],
    "solution": "awk '{total += $2; print $0, \"Total:\" total}' ledger.txt",
    "accepted_regex": [
      "awk\\s+.*total\\s*\\+=\\s*\\$2.*ledger\\.txt"
    ],
    "setup_files": {
      "ledger.txt": "deposit 100\nfee 5\ndeposit 50\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-43",
    "category": "Awk Processing",
    "title": "Filter Rows Where Two Columns Match",
    "difficulty": "Easy",
    "description": "Print lines from pairs.txt where column 1 equals column 2.",
    "objective": "awk '$1 == $2' pairs.txt",
    "hints": [
      "Command: awk '$1 == $2' pairs.txt"
    ],
    "solution": "awk '$1 == $2' pairs.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\$1\\s*==\\s*\\$2['\\\"]\\s+pairs\\.txt"
    ],
    "setup_files": {
      "pairs.txt": "foo bar\nfoo foo\ncat dog\napple apple\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-44",
    "category": "Awk Processing",
    "title": "Pass External Shell Variable into AWK with Flag -v",
    "difficulty": "Easy",
    "description": "Pass threshold=50 into awk via -v and print lines where field 2 is greater than threshold.",
    "objective": "awk -v threshold=50 '$2 > threshold' data.txt",
    "hints": [
      "Command: awk -v threshold=50 '$2 > threshold' data.txt"
    ],
    "solution": "awk -v threshold=50 '$2 > threshold' data.txt",
    "accepted_regex": [
      "awk\\s+-v\\s+threshold=50\\s+['\\\"]\\$2\\s*>\\s*threshold['\\\"]\\s+data\\.txt"
    ],
    "setup_files": {
      "data.txt": "a 10\nb 60\nc 45\nd 90\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-45",
    "category": "Awk Processing",
    "title": "Uppercase Field with toupper() Function",
    "difficulty": "Easy",
    "description": "Print all lines converting field 1 to uppercase using toupper().",
    "objective": "awk '{$1 = toupper($1); print}' names.txt",
    "hints": [
      "Command: awk '{$1 = toupper($1); print}' names.txt"
    ],
    "solution": "awk '{$1 = toupper($1); print}' names.txt",
    "accepted_regex": [
      "awk\\s+.*toupper.*names\\.txt"
    ],
    "setup_files": {
      "names.txt": "alice dev\nbob qa\ncharlie ops\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-46",
    "category": "Awk Processing",
    "title": "Lowercase Field with tolower() Function",
    "difficulty": "Easy",
    "description": "Print lines converting field 2 to lowercase using tolower().",
    "objective": "awk '{$2 = tolower($2); print}' email.txt",
    "hints": [
      "Command: awk '{$2 = tolower($2); print}' email.txt"
    ],
    "solution": "awk '{$2 = tolower($2); print}' email.txt",
    "accepted_regex": [
      "awk\\s+.*tolower.*email\\.txt"
    ],
    "setup_files": {
      "email.txt": "User1 JOHN@CORP.COM\nUser2 ALICE@CORP.COM\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-47",
    "category": "Awk Processing",
    "title": "Split Field into Sub-Array with split()",
    "difficulty": "Intermediate",
    "description": "Split the date field ($1 formatted as YYYY-MM-DD) by '-' and print the year and month.",
    "objective": "awk '{split($1, d, \"-\"); print d[1], d[2]}' dates.txt",
    "hints": [
      "Command: awk '{split($1, d, \"-\"); print d[1], d[2]}' dates.txt"
    ],
    "solution": "awk '{split($1, d, \"-\"); print d[1], d[2]}' dates.txt",
    "accepted_regex": [
      "awk\\s+.*split.*dates\\.txt"
    ],
    "setup_files": {
      "dates.txt": "2026-10-09 event1\n2025-05-12 event2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-48",
    "category": "Awk Processing",
    "title": "Print Only Lines Having Odd Number of Fields",
    "difficulty": "Intermediate",
    "description": "Filter lines in rows.txt to print only records having an odd number of fields (NF % 2 != 0).",
    "objective": "awk 'NF % 2 != 0' rows.txt",
    "hints": [
      "Command: awk 'NF % 2 != 0' rows.txt"
    ],
    "solution": "awk 'NF % 2 != 0' rows.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]NF\\s*%\\s*2\\s*(!=0|==1)['\\\"]\\s+rows\\.txt"
    ],
    "setup_files": {
      "rows.txt": "a b c\na b\na b c d e\na\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-49",
    "category": "Awk Processing",
    "title": "Compute Standard Variance in Column",
    "difficulty": "Advanced",
    "description": "Compute the difference between each row's value ($2) and the first row's baseline value.",
    "objective": "awk 'NR==1 {base=$2} {print $1, $2 - base}' baseline.txt",
    "hints": [
      "Command: awk 'NR==1 {base=$2} {print $1, $2 - base}' baseline.txt"
    ],
    "solution": "awk 'NR==1 {base=$2} {print $1, $2 - base}' baseline.txt",
    "accepted_regex": [
      "awk\\s+.*NR==1.*baseline\\.txt"
    ],
    "setup_files": {
      "baseline.txt": "point1 100\npoint2 105\npoint3 98\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-50",
    "category": "Awk Processing",
    "title": "Print Header Line Before Data Processing",
    "difficulty": "Easy",
    "description": "Print a formatted header 'USER | PID' using BEGIN before printing /etc/passwd users.",
    "objective": "awk -F: 'BEGIN {print \"USER | UID\"} {print $1, \"|\", $3}' passwd.sample",
    "hints": [
      "Command: awk -F: 'BEGIN {print \"USER | UID\"} {print $1, \"|\", $3}' passwd.sample"
    ],
    "solution": "awk -F: 'BEGIN {print \"USER | UID\"} {print $1, \"|\", $3}' passwd.sample",
    "accepted_regex": [
      "awk\\s+.*BEGIN.*passwd\\.sample"
    ],
    "setup_files": {
      "passwd.sample": "root:x:0:0:root:/root:/bin/bash\nbin:x:1:1:bin:/bin:/sbin/nologin\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-51",
    "category": "Awk Processing",
    "title": "Print Specific Range of Line Numbers (NR)",
    "difficulty": "Easy",
    "description": "Print lines 5 through 10 of book.txt using NR condition.",
    "objective": "awk 'NR>=5 && NR<=10' book.txt",
    "hints": [
      "Command: awk 'NR>=5 && NR<=10' book.txt"
    ],
    "solution": "awk 'NR>=5 && NR<=10' book.txt",
    "accepted_regex": [
      "awk\\s+['\\\"](NR>=5\\s*&&\\s*NR<=10|NR==5,NR==10)['\\\"]\\s+book\\.txt"
    ],
    "setup_files": {
      "book.txt": "line 1\nline 2\nline 3\nline 4\nline 5\nline 6\nline 7\nline 8\nline 9\nline 10\nline 11\nline 12\nline 13\nline 14\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-52",
    "category": "Awk Processing",
    "title": "Check If Column Value Is Within Numeric Range",
    "difficulty": "Easy",
    "description": "Filter products.txt to show lines where price ($2) is between 20 and 50 inclusive.",
    "objective": "awk '$2 >= 20 && $2 <= 50' products.txt",
    "hints": [
      "Command: awk '$2 >= 20 && $2 <= 50' products.txt"
    ],
    "solution": "awk '$2 >= 20 && $2 <= 50' products.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\$2\\s*>=\\s*20\\s*&&\\s*\\$2\\s*<=\\s*50['\\\"]\\s+products\\.txt"
    ],
    "setup_files": {
      "products.txt": "itemA 10\nitemB 25\nitemC 50\nitemD 75\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-53",
    "category": "Awk Processing",
    "title": "Filter Lines by Regex Pattern on Specific Field",
    "difficulty": "Easy",
    "description": "Print lines from services.txt where field 2 matches the regex ~ /^https?$/.",
    "objective": "awk '$2 ~ /^https?$/' services.txt",
    "hints": [
      "Command: awk '$2 ~ /^https?$/' services.txt"
    ],
    "solution": "awk '$2 ~ /^https?$/' services.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\$2\\s*~\\s*\\/(\\^https\\?\\$|https\\?)(\\/)?['\\\"]\\s+services\\.txt"
    ],
    "setup_files": {
      "services.txt": "web http\nsecure https\nftp ftp\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-54",
    "category": "Awk Processing",
    "title": "Invert Regex Match on Specific Field with !~",
    "difficulty": "Easy",
    "description": "Filter lines where field 3 does NOT match 'ACTIVE' using !~ operator.",
    "objective": "awk '$3 !~ /ACTIVE/' nodes.txt",
    "hints": [
      "Command: awk '$3 !~ /ACTIVE/' nodes.txt"
    ],
    "solution": "awk '$3 !~ /ACTIVE/' nodes.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]\\$3\\s*!~\\s*\\/ACTIVE\\/['\\\"]\\s+nodes\\.txt"
    ],
    "setup_files": {
      "nodes.txt": "node1 10.0.0.1 ACTIVE\nnode2 10.0.0.2 STANDBY\nnode3 10.0.0.3 DOWN\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-55",
    "category": "Awk Processing",
    "title": "Print Fields Starting from Field 3 to End of Line",
    "difficulty": "Intermediate",
    "description": "Print from column 3 through the end of line ($NF) omitting columns 1 and 2.",
    "objective": "awk '{for(i=3;i<=NF;i++) printf \"%s \", $i; print \"\"}' log.txt",
    "hints": [
      "Command: awk '{for(i=3;i<=NF;i++) printf \"%s \", $i; print \"\"}' log.txt"
    ],
    "solution": "awk '{for(i=3;i<=NF;i++) printf \"%s \", $i; print \"\"}' log.txt",
    "accepted_regex": [
      "awk\\s+.*for.*i=3.*log\\.txt"
    ],
    "setup_files": {
      "log.txt": "2026-10-09 12:00:00 User root performed sudo reboot\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-56",
    "category": "Awk Processing",
    "title": "Compute Percentage Distribution of Column 1",
    "difficulty": "Advanced",
    "description": "Read totals.txt twice (or store in memory) to output each row's percentage of overall sum.",
    "objective": "awk 'NR==FNR {total+=$2; next} {printf \"%s %.1f%%\\n\", $1, ($2/total)*100}' totals.txt totals.txt",
    "hints": [
      "Command: awk 'NR==FNR {total+=$2; next} {printf \"%s %.1f%%\\n\", $1, ($2/total)*100}' totals.txt totals.txt"
    ],
    "solution": "awk 'NR==FNR {total+=$2; next} {printf \"%s %.1f%%\\n\", $1, ($2/total)*100}' totals.txt totals.txt",
    "accepted_regex": [
      "awk\\s+.*total\\+=\\$2.*totals\\.txt"
    ],
    "setup_files": {
      "totals.txt": "deptA 50\ndeptB 150\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-57",
    "category": "Awk Processing",
    "title": "Print Unique Rows Preserving Original Order",
    "difficulty": "Intermediate",
    "description": "Remove duplicate lines from duplicates.txt keeping only the first occurrence in original order using !seen[$0]++.",
    "objective": "awk '!seen[$0]++' duplicates.txt",
    "hints": [
      "Command: awk '!seen[$0]++' duplicates.txt"
    ],
    "solution": "awk '!seen[$0]++' duplicates.txt",
    "accepted_regex": [
      "awk\\s+['\\\"]!seen\\[\\$0\\]\\+\\+['\\\"]\\s+duplicates\\.txt"
    ],
    "setup_files": {
      "duplicates.txt": "orange\napple\norange\nbanana\napple\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-58",
    "category": "Awk Processing",
    "title": "Join Two Files by Common Key Column",
    "difficulty": "Advanced",
    "description": "Join users.txt (id, name) and roles.txt (id, role) on field 1.",
    "objective": "awk 'NR==FNR {role[$1]=$2; next} $1 in role {print $1, $2, role[$1]}' roles.txt users.txt",
    "hints": [
      "Command: awk 'NR==FNR {role[$1]=$2; next} $1 in role {print $1, $2, role[$1]}' roles.txt users.txt"
    ],
    "solution": "awk 'NR==FNR {role[$1]=$2; next} $1 in role {print $1, $2, role[$1]}' roles.txt users.txt",
    "accepted_regex": [
      "awk\\s+.*NR==FNR.*roles\\.txt\\s+users\\.txt"
    ],
    "setup_files": {
      "roles.txt": "1 admin\n2 user\n",
      "users.txt": "1 Alice\n2 Bob\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-59",
    "category": "Awk Processing",
    "title": "Delete Trailing Spaces from All Records",
    "difficulty": "Easy",
    "description": "Strip trailing whitespace from every line using sub(/[[:space:]]+$/, \"\") in awk.",
    "objective": "awk '{sub(/[[:space:]]+$/, \"\"); print}' dirty.txt",
    "hints": [
      "Command: awk '{sub(/[[:space:]]+$/, \"\"); print}' dirty.txt"
    ],
    "solution": "awk '{sub(/[[:space:]]+$/, \"\"); print}' dirty.txt",
    "accepted_regex": [
      "awk\\s+.*sub.*dirty\\.txt"
    ],
    "setup_files": {
      "dirty.txt": "clean\nwith spaces    \n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "awk-60",
    "category": "Awk Processing",
    "title": "Print Line Number and Total Fields on Each Row",
    "difficulty": "Easy",
    "description": "Print 'Line NR has NF fields' for each row of matrix.txt.",
    "objective": "awk '{print \"Line\", NR, \"has\", NF, \"fields\"}' matrix.txt",
    "hints": [
      "Command: awk '{print \"Line\", NR, \"has\", NF, \"fields\"}' matrix.txt"
    ],
    "solution": "awk '{print \"Line\", NR, \"has\", NF, \"fields\"}' matrix.txt",
    "accepted_regex": [
      "awk\\s+.*NR.*NF.*matrix\\.txt"
    ],
    "setup_files": {
      "matrix.txt": "a b c\nd e\nf g h i\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-26",
    "category": "Find & Permissions",
    "title": "Find Files Modified in Past 24 Hours (-mtime 0)",
    "difficulty": "Easy",
    "description": "Search /var/log for files whose modification time is within the last 24 hours.",
    "objective": "find /var/log -type f -mtime 0",
    "hints": [
      "Command: find /var/log -type f -mtime 0"
    ],
    "solution": "find /var/log -type f -mtime 0",
    "accepted_regex": [
      "find\\s+\\/var\\/log\\s+.*-type\\s+f.*-mtime\\s+0"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-27",
    "category": "Find & Permissions",
    "title": "Find Files Accessed Within Past 30 Minutes (-amin)",
    "difficulty": "Intermediate",
    "description": "Search /tmp for files accessed within the last 30 minutes (-amin -30).",
    "objective": "find /tmp -type f -amin -30",
    "hints": [
      "Command: find /tmp -type f -amin -30"
    ],
    "solution": "find /tmp -type f -amin -30",
    "accepted_regex": [
      "find\\s+\\/tmp\\s+.*-type\\s+f.*-amin\\s+-30"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-28",
    "category": "Find & Permissions",
    "title": "Find Files Modified More Than 30 Days Ago",
    "difficulty": "Easy",
    "description": "Locate files under /archive modified more than 30 days ago (+30).",
    "objective": "find /archive -type f -mtime +30",
    "hints": [
      "Command: find /archive -type f -mtime +30"
    ],
    "solution": "find /archive -type f -mtime +30",
    "accepted_regex": [
      "find\\s+\\/archive\\s+.*-type\\s+f.*-mtime\\s+\\+30"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-29",
    "category": "Find & Permissions",
    "title": "Find Symbolic Links Pointing to Non-Existent Files (Broken Symlinks)",
    "difficulty": "Intermediate",
    "description": "Find broken symlinks in /etc using find's -xtype l test.",
    "objective": "find /etc -xtype l",
    "hints": [
      "Command: find /etc -xtype l"
    ],
    "solution": "find /etc -xtype l",
    "accepted_regex": [
      "find\\s+\\/etc\\s+.*-xtype\\s+l"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-30",
    "category": "Find & Permissions",
    "title": "Find Files Owned by Specific User ID",
    "difficulty": "Easy",
    "description": "Search /home for files owned by numeric UID 1000.",
    "objective": "find /home -uid 1000",
    "hints": [
      "Command: find /home -uid 1000"
    ],
    "solution": "find /home -uid 1000",
    "accepted_regex": [
      "find\\s+\\/home\\s+.*-uid\\s+1000"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-31",
    "category": "Find & Permissions",
    "title": "Find Files Owned by Specific Group ID",
    "difficulty": "Easy",
    "description": "Search /var for files belonging to numeric GID 50.",
    "objective": "find /var -gid 50",
    "hints": [
      "Command: find /var -gid 50"
    ],
    "solution": "find /var -gid 50",
    "accepted_regex": [
      "find\\s+\\/var\\s+.*-gid\\s+50"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-32",
    "category": "Find & Permissions",
    "title": "Find Empty Directories and Prune/Delete Them",
    "difficulty": "Intermediate",
    "description": "Find all empty directories under /tmp/scratch and delete them using -empty -delete.",
    "objective": "find /tmp/scratch -type d -empty -delete",
    "hints": [
      "Command: find /tmp/scratch -type d -empty -delete"
    ],
    "solution": "find /tmp/scratch -type d -empty -delete",
    "accepted_regex": [
      "find\\s+\\/tmp\\/scratch\\s+.*-type\\s+d.*-empty.*-delete"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-33",
    "category": "Find & Permissions",
    "title": "Find Files Newer Than Reference File (-newer)",
    "difficulty": "Intermediate",
    "description": "Find files under /data modified more recently than reference file /var/log/checkpoint.",
    "objective": "find /data -newer /var/log/checkpoint",
    "hints": [
      "Command: find /data -newer /var/log/checkpoint"
    ],
    "solution": "find /data -newer /var/log/checkpoint",
    "accepted_regex": [
      "find\\s+\\/data\\s+.*-newer\\s+\\/var\\/log\\/checkpoint"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-34",
    "category": "Find & Permissions",
    "title": "Limit Find Search Depth to Current Directory Only",
    "difficulty": "Easy",
    "description": "Search only the immediate top level of /var/log without recursing into subdirectories (-maxdepth 1).",
    "objective": "find /var/log -maxdepth 1 -type f",
    "hints": [
      "Command: find /var/log -maxdepth 1 -type f"
    ],
    "solution": "find /var/log -maxdepth 1 -type f",
    "accepted_regex": [
      "find\\s+\\/var\\/log\\s+.*-maxdepth\\s+1.*-type\\s+f"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-35",
    "category": "Find & Permissions",
    "title": "Restrict Find Search by Minimum Depth (-mindepth)",
    "difficulty": "Intermediate",
    "description": "Search /srv for directories starting at depth 2 or deeper (-mindepth 2).",
    "objective": "find /srv -mindepth 2 -type d",
    "hints": [
      "Command: find /srv -mindepth 2 -type d"
    ],
    "solution": "find /srv -mindepth 2 -type d",
    "accepted_regex": [
      "find\\s+\\/srv\\s+.*-mindepth\\s+2.*-type\\s+d"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-36",
    "category": "Find & Permissions",
    "title": "Find Files with Exact 3 Hard Links",
    "difficulty": "Intermediate",
    "description": "Locate files under /data having exactly 3 hard link references (-links 3).",
    "objective": "find /data -type f -links 3",
    "hints": [
      "Command: find /data -type f -links 3"
    ],
    "solution": "find /data -type f -links 3",
    "accepted_regex": [
      "find\\s+\\/data\\s+.*-type\\s+f.*-links\\s+3"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-37",
    "category": "Find & Permissions",
    "title": "Find Block Special Device Files",
    "difficulty": "Easy",
    "description": "Search /dev for block device nodes (-type b).",
    "objective": "find /dev -type b",
    "hints": [
      "Command: find /dev -type b"
    ],
    "solution": "find /dev -type b",
    "accepted_regex": [
      "find\\s+\\/dev\\s+.*-type\\s+b"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-38",
    "category": "Find & Permissions",
    "title": "Find Character Special Device Files",
    "difficulty": "Easy",
    "description": "Search /dev for character device nodes (-type c).",
    "objective": "find /dev -type c",
    "hints": [
      "Command: find /dev -type c"
    ],
    "solution": "find /dev -type c",
    "accepted_regex": [
      "find\\s+\\/dev\\s+.*-type\\s+c"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-39",
    "category": "Find & Permissions",
    "title": "Find Named Pipe FIFO Files",
    "difficulty": "Easy",
    "description": "Search /run for named pipe FIFO files (-type p).",
    "objective": "find /run -type p",
    "hints": [
      "Command: find /run -type p"
    ],
    "solution": "find /run -type p",
    "accepted_regex": [
      "find\\s+\\/run\\s+.*-type\\s+p"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-40",
    "category": "Find & Permissions",
    "title": "Find Unix Domain Socket Files",
    "difficulty": "Easy",
    "description": "Search /run for active Unix domain socket files (-type s).",
    "objective": "find /run -type s",
    "hints": [
      "Command: find /run -type s"
    ],
    "solution": "find /run -type s",
    "accepted_regex": [
      "find\\s+\\/run\\s+.*-type\\s+s"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-41",
    "category": "Find & Permissions",
    "title": "Find Files Without Descending Past Mount Points (-xdev)",
    "difficulty": "Intermediate",
    "description": "Search root filesystem '/' for files > 1GB without crossing into mounted filesystems (-xdev).",
    "objective": "find / -xdev -type f -size +1G",
    "hints": [
      "Command: find / -xdev -type f -size +1G"
    ],
    "solution": "find / -xdev -type f -size +1G",
    "accepted_regex": [
      "find\\s+\\/\\s+.*-xdev.*-type\\s+f.*-size\\s+\\+1G"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-42",
    "category": "Find & Permissions",
    "title": "Set Default POSIX Access Control List on Directory",
    "difficulty": "Intermediate",
    "description": "Configure default ACL so group 'developers' receives rwx on newly created files in /shared/project.",
    "objective": "setfacl -d -m g:developers:rwx /shared/project",
    "hints": [
      "Command: setfacl -d -m g:developers:rwx /shared/project"
    ],
    "solution": "setfacl -d -m g:developers:rwx /shared/project",
    "accepted_regex": [
      "setfacl\\s+.*-d.*-m\\s+g:developers:rwx\\s+\\/shared\\/project"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-43",
    "category": "Find & Permissions",
    "title": "Remove Specific User Entry from File ACL",
    "difficulty": "Intermediate",
    "description": "Remove user 'bob' from the ACL entries of secure.dat using setfacl -x.",
    "objective": "setfacl -x u:bob secure.dat",
    "hints": [
      "Command: setfacl -x u:bob secure.dat"
    ],
    "solution": "setfacl -x u:bob secure.dat",
    "accepted_regex": [
      "setfacl\\s+-x\\s+u:bob\\s+secure\\.dat"
    ],
    "setup_files": {
      "secure.dat": "secret data\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-44",
    "category": "Find & Permissions",
    "title": "Remove All Extended ACL Entries Recursively",
    "difficulty": "Intermediate",
    "description": "Completely remove all extended ACL entries from /shared/project recursively using setfacl -b -R.",
    "objective": "setfacl -R -b /shared/project",
    "hints": [
      "Command: setfacl -R -b /shared/project"
    ],
    "solution": "setfacl -R -b /shared/project",
    "accepted_regex": [
      "setfacl\\s+(-R\\s+-b|-b\\s+-R|-Rb|-bR)\\s+\\/shared\\/project"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-45",
    "category": "Find & Permissions",
    "title": "Inspect File Access Control List with getfacl",
    "difficulty": "Easy",
    "description": "View POSIX ACL entries on /shared/data using getfacl.",
    "objective": "getfacl /shared/data",
    "hints": [
      "Command: getfacl /shared/data"
    ],
    "solution": "getfacl /shared/data",
    "accepted_regex": [
      "getfacl\\s+\\/shared\\/data"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-46",
    "category": "Find & Permissions",
    "title": "Set Linux File Immutable Attribute with chattr",
    "difficulty": "Intermediate",
    "description": "Prevent /etc/resolv.conf from being modified, overwritten, or deleted using chattr +i.",
    "objective": "chattr +i /etc/resolv.conf",
    "hints": [
      "Command: chattr +i /etc/resolv.conf"
    ],
    "solution": "chattr +i /etc/resolv.conf",
    "accepted_regex": [
      "chattr\\s+\\+i\\s+\\/etc\\/resolv\\.conf"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-47",
    "category": "Find & Permissions",
    "title": "Remove Immutable Attribute with chattr",
    "difficulty": "Intermediate",
    "description": "Remove the immutable attribute from /etc/resolv.conf so it can be updated again.",
    "objective": "chattr -i /etc/resolv.conf",
    "hints": [
      "Command: chattr -i /etc/resolv.conf"
    ],
    "solution": "chattr -i /etc/resolv.conf",
    "accepted_regex": [
      "chattr\\s+-i\\s+\\/etc\\/resolv\\.conf"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-48",
    "category": "Find & Permissions",
    "title": "Set Append-Only Attribute with chattr",
    "difficulty": "Intermediate",
    "description": "Configure log file /var/log/audit.custom to be append-only using chattr +a.",
    "objective": "chattr +a /var/log/audit.custom",
    "hints": [
      "Command: chattr +a /var/log/audit.custom"
    ],
    "solution": "chattr +a /var/log/audit.custom",
    "accepted_regex": [
      "chattr\\s+\\+a\\s+\\/var\\/log\\/audit\\.custom"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-49",
    "category": "Find & Permissions",
    "title": "List Ext4/XFS File Attributes with lsattr",
    "difficulty": "Easy",
    "description": "Inspect file attributes on /etc/shadow using lsattr.",
    "objective": "lsattr /etc/shadow",
    "hints": [
      "Command: lsattr /etc/shadow"
    ],
    "solution": "lsattr /etc/shadow",
    "accepted_regex": [
      "lsattr\\s+\\/etc\\/shadow"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-50",
    "category": "Find & Permissions",
    "title": "Find Files Without Read Permission for Current User",
    "difficulty": "Intermediate",
    "description": "Find files under /secure that are not readable by the current user (! -readable).",
    "objective": "find /secure ! -readable",
    "hints": [
      "Command: find /secure ! -readable"
    ],
    "solution": "find /secure ! -readable",
    "accepted_regex": [
      "find\\s+\\/secure\\s+.*!\\s+-readable"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-51",
    "category": "Find & Permissions",
    "title": "Find Files That Are World-Writable",
    "difficulty": "Intermediate",
    "description": "Audit security finding all files under /var/tmp that have world-writable permission (perm -002).",
    "objective": "find /var/tmp -type f -perm -002",
    "hints": [
      "Command: find /var/tmp -type f -perm -002"
    ],
    "solution": "find /var/tmp -type f -perm -002",
    "accepted_regex": [
      "find\\s+\\/var\\/tmp\\s+.*-type\\s+f.*-perm\\s+(-002|-0002)"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-52",
    "category": "Find & Permissions",
    "title": "Change Group Ownership Recursively",
    "difficulty": "Easy",
    "description": "Change group ownership of /opt/app to 'wheel' recursively.",
    "objective": "chgrp -R wheel /opt/app",
    "hints": [
      "Command: chgrp -R wheel /opt/app"
    ],
    "solution": "chgrp -R wheel /opt/app",
    "accepted_regex": [
      "chgrp\\s+-R\\s+wheel\\s+\\/opt\\/app"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-53",
    "category": "Find & Permissions",
    "title": "Change Owner and Group Simultaneously",
    "difficulty": "Easy",
    "description": "Set user 'nginx' and group 'nginx' on /usr/share/nginx/html recursively.",
    "objective": "chown -R nginx:nginx /usr/share/nginx/html",
    "hints": [
      "Command: chown -R nginx:nginx /usr/share/nginx/html"
    ],
    "solution": "chown -R nginx:nginx /usr/share/nginx/html",
    "accepted_regex": [
      "chown\\s+-R\\s+nginx:nginx\\s+\\/usr\\/share\\/nginx\\/html"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-54",
    "category": "Find & Permissions",
    "title": "Preserve Symlink Targets During Chown",
    "difficulty": "Intermediate",
    "description": "Change ownership of symlink itself (/opt/link) without affecting target using chown -h.",
    "objective": "chown -h appuser:appgroup /opt/link",
    "hints": [
      "Command: chown -h appuser:appgroup /opt/link"
    ],
    "solution": "chown -h appuser:appgroup /opt/link",
    "accepted_regex": [
      "chown\\s+-h\\s+appuser:appgroup\\s+\\/opt\\/link"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-55",
    "category": "Find & Permissions",
    "title": "Copy Permissions from Reference File with Chmod --reference",
    "difficulty": "Intermediate",
    "description": "Apply the exact same permissions from template.conf to new.conf.",
    "objective": "chmod --reference=template.conf new.conf",
    "hints": [
      "Command: chmod --reference=template.conf new.conf"
    ],
    "solution": "chmod --reference=template.conf new.conf",
    "accepted_regex": [
      "chmod\\s+--reference=template\\.conf\\s+new\\.conf"
    ],
    "setup_files": {
      "template.conf": "content\n",
      "new.conf": "content\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-56",
    "category": "Find & Permissions",
    "title": "Display Octal Permissions with Stat",
    "difficulty": "Easy",
    "description": "Print only the 3/4-digit octal permission string of file script.sh using stat -c '%a'.",
    "objective": "stat -c '%a' script.sh",
    "hints": [
      "Command: stat -c '%a' script.sh"
    ],
    "solution": "stat -c '%a' script.sh",
    "accepted_regex": [
      "stat\\s+(-c\\s+['\\\"]?%a['\\\"]?|--format=['\\\"]?%a['\\\"]?)\\s+script\\.sh"
    ],
    "setup_files": {
      "script.sh": "#!/bin/bash\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-57",
    "category": "Find & Permissions",
    "title": "Find Files Modified Between Specific Timestamps",
    "difficulty": "Advanced",
    "description": "Find files under /var/log modified newer than '2026-10-01' but older than '2026-10-05' using -newermt.",
    "objective": "find /var/log -newermt '2026-10-01' ! -newermt '2026-10-05'",
    "hints": [
      "Command: find /var/log -newermt '2026-10-01' ! -newermt '2026-10-05'"
    ],
    "solution": "find /var/log -newermt '2026-10-01' ! -newermt '2026-10-05'",
    "accepted_regex": [
      "find\\s+\\/var\\/log\\s+.*-newermt\\s+['\\\"]?2026-10-01.*!\\s+-newermt\\s+['\\\"]?2026-10-05"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-58",
    "category": "Find & Permissions",
    "title": "Find Files Matching Regex Path Pattern",
    "difficulty": "Intermediate",
    "description": "Find files under /data whose full path matches regex '.*\\.bak[0-9]+' using -regex.",
    "objective": "find /data -regex '.*\\.bak[0-9]+'",
    "hints": [
      "Command: find /data -regex '.*\\.bak[0-9]+'"
    ],
    "solution": "find /data -regex '.*\\.bak[0-9]+'",
    "accepted_regex": [
      "find\\s+\\/data\\s+.*-regex\\s+['\\\"].*\\.bak\\[0-9\\]\\+['\\\"]"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-59",
    "category": "Find & Permissions",
    "title": "Execute Batch Command on Found Files with Plus Terminator",
    "difficulty": "Intermediate",
    "description": "Run chmod 644 on all found *.txt files under /docs using -exec ... + (batch invocation).",
    "objective": "find /docs -type f -name '*.txt' -exec chmod 644 {} +",
    "hints": [
      "Command: find /docs -type f -name '*.txt' -exec chmod 644 {} +"
    ],
    "solution": "find /docs -type f -name '*.txt' -exec chmod 644 {} +",
    "accepted_regex": [
      "find\\s+\\/docs\\s+.*-exec\\s+chmod\\s+644\\s+\\{\\}\\s+\\+"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "find-60",
    "category": "Find & Permissions",
    "title": "Find and Delete Stale Session Files Interactively",
    "difficulty": "Intermediate",
    "description": "Locate files under /tmp with extension '.sess' older than 7 days and delete them with -delete.",
    "objective": "find /tmp -type f -name '*.sess' -mtime +7 -delete",
    "hints": [
      "Command: find /tmp -type f -name '*.sess' -mtime +7 -delete"
    ],
    "solution": "find /tmp -type f -name '*.sess' -mtime +7 -delete",
    "accepted_regex": [
      "find\\s+\\/tmp\\s+.*-type\\s+f.*-name\\s+['\\\"]?\\*\\.sess['\\\"].*-mtime\\s+\\+7.*-delete"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-31",
    "category": "Pipes & Redirections",
    "title": "Redirect Standard Output and Error to Separate Files",
    "difficulty": "Easy",
    "description": "Execute './build.sh' saving stdout to build.log and stderr to error.log simultaneously.",
    "objective": "./build.sh > build.log 2> error.log",
    "hints": [
      "Command: ./build.sh > build.log 2> error.log"
    ],
    "solution": "./build.sh > build.log 2> error.log",
    "accepted_regex": [
      "\\.\\/build\\.sh\\s+>\\s*build\\.log\\s+2>\\s*error\\.log"
    ],
    "setup_files": {
      "build.sh": "#!/bin/bash\necho ok\n"
    },
    "setup_perms": {
      "build.sh": 493
    },
    "verify_cmd": ""
  },
  {
    "id": "pipe-32",
    "category": "Pipes & Redirections",
    "title": "Append Both Stdout and Stderr to Same File",
    "difficulty": "Easy",
    "description": "Append both stdout and stderr of './task.sh' to log.txt using &>>.",
    "objective": "./task.sh &>> log.txt",
    "hints": [
      "Command: ./task.sh &>> log.txt"
    ],
    "solution": "./task.sh &>> log.txt",
    "accepted_regex": [
      "\\.\\/task\\.sh\\s+(&>>|>>\\s*log\\.txt\\s+2>&1)\\s*log\\.txt"
    ],
    "setup_files": {
      "task.sh": "#!/bin/bash\necho done\n"
    },
    "setup_perms": {
      "task.sh": 493
    },
    "verify_cmd": ""
  },
  {
    "id": "pipe-33",
    "category": "Pipes & Redirections",
    "title": "Discard All Output to Null Device",
    "difficulty": "Easy",
    "description": "Run noisy command 'cleanup.sh' silencing both stdout and stderr by redirecting to /dev/null.",
    "objective": "./cleanup.sh > /dev/null 2>&1",
    "hints": [
      "Command: ./cleanup.sh > /dev/null 2>&1"
    ],
    "solution": "./cleanup.sh > /dev/null 2>&1",
    "accepted_regex": [
      "\\.\\/cleanup\\.sh\\s+(>\\s*\\/dev\\/null\\s+2>&1|&>\\s*\\/dev\\/null)"
    ],
    "setup_files": {
      "cleanup.sh": "#!/bin/bash\necho noisy\n"
    },
    "setup_perms": {
      "cleanup.sh": 493
    },
    "verify_cmd": ""
  },
  {
    "id": "pipe-34",
    "category": "Pipes & Redirections",
    "title": "Process Substitution Output Comparison with Diff",
    "difficulty": "Intermediate",
    "description": "Compare sorted output of list1.txt and list2.txt without temporary files using process substitution <(...).",
    "objective": "diff <(sort list1.txt) <(sort list2.txt)",
    "hints": [
      "Command: diff <(sort list1.txt) <(sort list2.txt)"
    ],
    "solution": "diff <(sort list1.txt) <(sort list2.txt)",
    "accepted_regex": [
      "diff\\s+<\\(sort\\s+list1\\.txt\\)\\s+<\\(sort\\s+list2\\.txt\\)"
    ],
    "setup_files": {
      "list1.txt": "b\na\n",
      "list2.txt": "a\nc\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-35",
    "category": "Pipes & Redirections",
    "title": "Feed Command Output into Loop with Process Substitution",
    "difficulty": "Advanced",
    "description": "Read lines from process substitution into a while loop: while read line; do echo \"Line: $line\"; done < <(cat items.txt).",
    "objective": "while read line; do echo \"$line\"; done < <(cat items.txt)",
    "hints": [
      "Command: while read line; do echo \"$line\"; done < <(cat items.txt)"
    ],
    "solution": "while read line; do echo \"$line\"; done < <(cat items.txt)",
    "accepted_regex": [
      "while\\s+read\\s+line;.*<\\s*<\\(cat\\s+items\\.txt\\)"
    ],
    "setup_files": {
      "items.txt": "item1\nitem2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-36",
    "category": "Pipes & Redirections",
    "title": "Tee Command Appending to Root Owned File with Sudo",
    "difficulty": "Intermediate",
    "description": "Echo '10.0.0.1 db.local' and append it to /etc/hosts using sudo tee -a.",
    "objective": "echo '10.0.0.1 db.local' | sudo tee -a /etc/hosts",
    "hints": [
      "Command: echo '10.0.0.1 db.local' | sudo tee -a /etc/hosts"
    ],
    "solution": "echo '10.0.0.1 db.local' | sudo tee -a /etc/hosts",
    "accepted_regex": [
      "echo\\s+['\\\"]10\\.0\\.0\\.1\\s+db\\.local['\\\"]\\s*\\|\\s*sudo\\s+tee\\s+-a\\s+\\/etc\\/hosts"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-37",
    "category": "Pipes & Redirections",
    "title": "Pass Multiline String with Here-String (<<<)",
    "difficulty": "Easy",
    "description": "Pass string 'alpha beta gamma' into tr to replace spaces with newlines using here-string <<<.",
    "objective": "tr ' ' '\\n' <<< 'alpha beta gamma'",
    "hints": [
      "Command: tr ' ' '\\n' <<< 'alpha beta gamma'"
    ],
    "solution": "tr ' ' '\\n' <<< 'alpha beta gamma'",
    "accepted_regex": [
      "tr\\s+.*<<<\\s*['\\\"]alpha beta gamma['\\\"]"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-38",
    "category": "Pipes & Redirections",
    "title": "Parallel Command Execution with xargs -P",
    "difficulty": "Intermediate",
    "description": "Download URLs listed in urls.txt in parallel using up to 4 worker processes with xargs -n 1 -P 4 curl -O.",
    "objective": "cat urls.txt | xargs -n 1 -P 4 curl -O",
    "hints": [
      "Command: cat urls.txt | xargs -n 1 -P 4 curl -O or xargs -n 1 -P 4 curl -O < urls.txt"
    ],
    "solution": "cat urls.txt | xargs -n 1 -P 4 curl -O",
    "accepted_regex": [
      ".*xargs\\s+.*-P\\s*4.*curl\\s+-O"
    ],
    "setup_files": {
      "urls.txt": "http://example.com/1\nhttp://example.com/2\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-39",
    "category": "Pipes & Redirections",
    "title": "Find Inode Duplicates Using Sort and Uniq",
    "difficulty": "Intermediate",
    "description": "List sorted unique words and their occurrence counts in essay.txt using sort | uniq -c | sort -nr.",
    "objective": "sort essay.txt | uniq -c | sort -nr",
    "hints": [
      "Command: sort essay.txt | uniq -c | sort -nr"
    ],
    "solution": "sort essay.txt | uniq -c | sort -nr",
    "accepted_regex": [
      "sort\\s+essay\\.txt\\s*\\|\\s*uniq\\s+-c\\s*\\|\\s*sort\\s+-nr"
    ],
    "setup_files": {
      "essay.txt": "the\nlinux\nthe\nrhel\nthe\nlinux\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-40",
    "category": "Pipes & Redirections",
    "title": "Find Lines Common to Both Sorted Files with Comm",
    "difficulty": "Intermediate",
    "description": "Print lines that are common to both fileA.sorted and fileB.sorted using comm -12.",
    "objective": "comm -12 fileA.sorted fileB.sorted",
    "hints": [
      "Command: comm -12 fileA.sorted fileB.sorted"
    ],
    "solution": "comm -12 fileA.sorted fileB.sorted",
    "accepted_regex": [
      "comm\\s+-12\\s+fileA\\.sorted\\s+fileB\\.sorted"
    ],
    "setup_files": {
      "fileA.sorted": "apple\nbanana\norange\n",
      "fileB.sorted": "banana\ngrape\norange\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-41",
    "category": "Pipes & Redirections",
    "title": "Find Lines Unique to First File Only with Comm",
    "difficulty": "Intermediate",
    "description": "Print lines that exist only in fileA.sorted and not in fileB.sorted using comm -23.",
    "objective": "comm -23 fileA.sorted fileB.sorted",
    "hints": [
      "Command: comm -23 fileA.sorted fileB.sorted"
    ],
    "solution": "comm -23 fileA.sorted fileB.sorted",
    "accepted_regex": [
      "comm\\s+-23\\s+fileA\\.sorted\\s+fileB\\.sorted"
    ],
    "setup_files": {
      "fileA.sorted": "apple\nbanana\n",
      "fileB.sorted": "banana\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-42",
    "category": "Pipes & Redirections",
    "title": "Split Large File into 1000-Line Chunks",
    "difficulty": "Easy",
    "description": "Split big.log into smaller files of at most 1000 lines each with prefix 'chunk_'.",
    "objective": "split -l 1000 big.log chunk_",
    "hints": [
      "Command: split -l 1000 big.log chunk_"
    ],
    "solution": "split -l 1000 big.log chunk_",
    "accepted_regex": [
      "split\\s+-l\\s+1000\\s+big\\.log\\s+chunk_"
    ],
    "setup_files": {
      "big.log": "line\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\nline\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-43",
    "category": "Pipes & Redirections",
    "title": "Merge Corresponding Lines Side-by-Side with Paste",
    "difficulty": "Easy",
    "description": "Merge lines from names.txt and numbers.txt separated by a tab using paste.",
    "objective": "paste names.txt numbers.txt",
    "hints": [
      "Command: paste names.txt numbers.txt"
    ],
    "solution": "paste names.txt numbers.txt",
    "accepted_regex": [
      "paste\\s+names\\.txt\\s+numbers\\.txt"
    ],
    "setup_files": {
      "names.txt": "Alice\nBob\n",
      "numbers.txt": "100\n200\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-44",
    "category": "Pipes & Redirections",
    "title": "Join Two Files on Matching Field with Join Command",
    "difficulty": "Intermediate",
    "description": "Join sorted files f1.txt and f2.txt on the first field using join.",
    "objective": "join f1.txt f2.txt",
    "hints": [
      "Command: join f1.txt f2.txt"
    ],
    "solution": "join f1.txt f2.txt",
    "accepted_regex": [
      "join\\s+f1\\.txt\\s+f2\\.txt"
    ],
    "setup_files": {
      "f1.txt": "1 Alice\n2 Bob\n",
      "f2.txt": "1 HR\n2 IT\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-45",
    "category": "Pipes & Redirections",
    "title": "Filter Duplicated Consecutive Lines with Uniq -d",
    "difficulty": "Easy",
    "description": "Print only repeated duplicate lines from sorted_list.txt using uniq -d.",
    "objective": "uniq -d sorted_list.txt",
    "hints": [
      "Command: uniq -d sorted_list.txt"
    ],
    "solution": "uniq -d sorted_list.txt",
    "accepted_regex": [
      "uniq\\s+-d\\s+sorted_list\\.txt"
    ],
    "setup_files": {
      "sorted_list.txt": "a\nb\nb\nc\nd\nd\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-46",
    "category": "Pipes & Redirections",
    "title": "Filter Only Unique (Non-Repeated) Lines with Uniq -u",
    "difficulty": "Easy",
    "description": "Print only lines that appear exactly once in sorted_list.txt using uniq -u.",
    "objective": "uniq -u sorted_list.txt",
    "hints": [
      "Command: uniq -u sorted_list.txt"
    ],
    "solution": "uniq -u sorted_list.txt",
    "accepted_regex": [
      "uniq\\s+-u\\s+sorted_list\\.txt"
    ],
    "setup_files": {
      "sorted_list.txt": "a\nb\nb\nc\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-47",
    "category": "Pipes & Redirections",
    "title": "Reverse Line Characters Horizontally with Rev",
    "difficulty": "Easy",
    "description": "Reverse characters on every line of palindrome.txt using rev.",
    "objective": "rev palindrome.txt",
    "hints": [
      "Command: rev palindrome.txt"
    ],
    "solution": "rev palindrome.txt",
    "accepted_regex": [
      "rev\\s+palindrome\\.txt"
    ],
    "setup_files": {
      "palindrome.txt": "racecar\nstep on no pets\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-48",
    "category": "Pipes & Redirections",
    "title": "Sort Lines in Numerical Human-Readable Format",
    "difficulty": "Easy",
    "description": "Sort sizes.txt containing human-readable values (10M, 2G, 500K) using sort -h.",
    "objective": "sort -h sizes.txt",
    "hints": [
      "Command: sort -h sizes.txt"
    ],
    "solution": "sort -h sizes.txt",
    "accepted_regex": [
      "sort\\s+-h\\s+sizes\\.txt"
    ],
    "setup_files": {
      "sizes.txt": "2G\n500K\n10M\n1G\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-49",
    "category": "Pipes & Redirections",
    "title": "Translate and Squeeze Consecutive Repeating Characters",
    "difficulty": "Easy",
    "description": "Squeeze multiple consecutive spaces into single space characters in spaces.txt using tr -s ' '.",
    "objective": "tr -s ' ' < spaces.txt",
    "hints": [
      "Command: tr -s ' ' < spaces.txt"
    ],
    "solution": "tr -s ' ' < spaces.txt",
    "accepted_regex": [
      "tr\\s+-s\\s+['\\\"] ['\\\"]\\s*<\\s*spaces\\.txt",
      "cat\\s+spaces\\.txt\\s*\\|\\s*tr\\s+-s\\s+['\\\"] ['\\\"]"
    ],
    "setup_files": {
      "spaces.txt": "hello    world    here\n"
    },
    "setup_perms": {},
    "verify_cmd": ""
  },
  {
    "id": "pipe-50",
    "category": "Pipes & Redirections",
    "title": "Pass Null-Delimited File List into Tar",
    "difficulty": "Intermediate",
    "description": "Find *.conf files in /etc and archive them to /tmp/configs.tar using find -print0 and tar --null -T -.",
    "objective": "find /etc -name '*.conf' -print0 | tar --null -T - -cvf /tmp/configs.tar",
    "hints": [
      "Command: find /etc -name '*.conf' -print0 | tar --null -T - -cvf /tmp/configs.tar"
    ],
    "solution": "find /etc -name '*.conf' -print0 | tar --null -T - -cvf /tmp/configs.tar",
    "accepted_regex": [
      "find\\s+\\/etc\\s+.*-print0\\s*\\|\\s*tar\\s+.*--null.*-T\\s+-.*\\/tmp\\/configs\\.tar"
    ],
    "setup_files": {},
    "setup_perms": {},
    "verify_cmd": ""
  }
]''')

SCENARIOS = list(BUILTIN_SCENARIOS)

# ----------------------------------------------------------------------
# Terminal Colors & Theme
# ----------------------------------------------------------------------
COLOR_RED_HAT = 1
COLOR_SUCCESS = 2
COLOR_ERROR = 3
COLOR_WARN = 4
COLOR_CYAN = 5
COLOR_DIM = 6
COLOR_HEADER = 7

def init_colors():
    curses.start_color()
    curses.use_default_colors()
    try:
        curses.init_pair(COLOR_RED_HAT, curses.COLOR_RED, -1)
        curses.init_pair(COLOR_SUCCESS, curses.COLOR_GREEN, -1)
        curses.init_pair(COLOR_ERROR, curses.COLOR_RED, -1)
        curses.init_pair(COLOR_WARN, curses.COLOR_YELLOW, -1)
        curses.init_pair(COLOR_CYAN, curses.COLOR_CYAN, -1)
        curses.init_pair(COLOR_DIM, curses.COLOR_WHITE, -1)
        curses.init_pair(COLOR_HEADER, curses.COLOR_WHITE, curses.COLOR_RED)
    except Exception:
        pass


# ----------------------------------------------------------------------
# Execution Engine (Real RHEL 9 Snapshot Execution + Scratch Sandboxing)
# ----------------------------------------------------------------------
class RHELExecutionEngine:
    def __init__(self):
        self.scratch_dir = tempfile.mkdtemp(prefix="shell_drill_rhel9_")

    def cleanup(self):
        try:
            shutil.rmtree(self.scratch_dir, ignore_errors=True)
        except Exception:
            pass

    def setup_scenario(self, scenario):
        for item in os.listdir(self.scratch_dir):
            item_path = os.path.join(self.scratch_dir, item)
            try:
                if os.path.isdir(item_path):
                    shutil.rmtree(item_path)
                else:
                    os.unlink(item_path)
            except Exception:
                pass

        files = scenario.get("setup_files", {})
        for rel_path, file_data in files.items():
            full_path = os.path.join(self.scratch_dir, rel_path)
            os.makedirs(os.path.dirname(full_path), exist_ok=True)
            if isinstance(file_data, str) and file_data.startswith("__SPARSE_BYTES:"):
                sparse_size = int(file_data.split(":")[1])
                with open(full_path, "wb") as f:
                    f.truncate(sparse_size)
            else:
                with open(full_path, "w", encoding="utf-8") as f:
                    f.write(file_data)

        perms = scenario.get("setup_perms", {})
        for rel_path, mode in perms.items():
            full_path = os.path.join(self.scratch_dir, rel_path)
            if os.path.exists(full_path):
                try:
                    os.chmod(full_path, mode)
                except Exception:
                    pass

    def execute(self, cmd_str, scenario):
        clean_cmd = cmd_str.strip()
        if not clean_cmd:
            return False, "", "Empty command entered."

        regex_matched = False
        for pattern in scenario.get("accepted_regex", []):
            if re.search(pattern, clean_cmd):
                regex_matched = True
                break

        try:
            proc = subprocess.run(
                clean_cmd,
                shell=True,
                cwd=self.scratch_dir,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
                timeout=5.0,
                executable="/bin/bash"
            )
            user_stdout = proc.stdout
            user_stderr = proc.stderr
            combined_output = user_stdout + (f"\n[STDERR] {user_stderr}" if user_stderr else "")

            # If scenario defines a verification command
            if "verify_cmd" in scenario and scenario["verify_cmd"]:
                v_proc = subprocess.run(
                    scenario["verify_cmd"],
                    shell=True,
                    cwd=self.scratch_dir,
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE,
                    text=True,
                    timeout=3.0,
                    executable="/bin/bash"
                )
                if v_proc.returncode == 0 or regex_matched:
                    return True, combined_output, "VALIDATED! RHEL 9 system state satisfies scenario."

            # File redirection check (e.g. audit.log)
            if "audit.log" in clean_cmd:
                target_file = os.path.join(self.scratch_dir, "audit.log")
                if os.path.exists(target_file):
                    with open(target_file, "r") as f:
                        c = f.read()
                        if "Missing database credentials" in c and "Deploying artifacts" in c:
                            return True, combined_output, "EXACT MATCH! Both stdout and stderr captured."

            # Reference output comparison for stream/grep/sed/awk tasks
            if "setup_files" in scenario and not regex_matched:
                ref_dir = tempfile.mkdtemp(prefix="shell_drill_ref_")
                try:
                    for item in os.listdir(self.scratch_dir):
                        s_p = os.path.join(self.scratch_dir, item)
                        d_p = os.path.join(ref_dir, item)
                        if os.path.isdir(s_p):
                            shutil.copytree(s_p, d_p)
                        else:
                            shutil.copy2(s_p, d_p)
                    ref_proc = subprocess.run(
                        scenario["solution"],
                        shell=True,
                        cwd=ref_dir,
                        stdout=subprocess.PIPE,
                        stderr=subprocess.PIPE,
                        text=True,
                        timeout=5.0,
                        executable="/bin/bash"
                    )
                    if user_stdout.strip() == ref_proc.stdout.strip() and proc.returncode == 0:
                        return True, combined_output, "EXACT MATCH! Output matches target."
                finally:
                    shutil.rmtree(ref_dir, ignore_errors=True)

            if regex_matched:
                return True, combined_output, "VALIDATED! Correct RHEL command syntax and arguments."

            msg = f"Command exited with code {proc.returncode}."
            if proc.returncode != 0 and user_stderr:
                msg += f" Error: {user_stderr.strip()[:60]}"
            else:
                msg += " Target objective not satisfied. Check hint."
            return False, combined_output, msg

        except subprocess.TimeoutExpired:
            return False, "", "Execution timed out (5.0s limit)."
        except Exception as e:
            return False, "", f"Execution error: {str(e)}"


# ----------------------------------------------------------------------
# Curses TUI Interface
# ----------------------------------------------------------------------
class ShellDrillTUI:
    def __init__(self, stdscr, category_filter=None, session_count=None):
        self.stdscr = stdscr
        self.engine = RHELExecutionEngine()
        self.current_idx = 0
        self.score = 0
        self.streak = 0
        self.max_streak = 0
        self.attempts = 0
        self.successes = 0
        self.session_completed = 0
        self.session_count = session_count  # None means infinite / all
        self.history = []
        self.history_idx = 0
        self.user_cmd = ""
        self.cursor_pos = 0
        self.feedback = None
        self.feedback_success = False
        self.feedback_output = ""
        self.hint_level = 0
        self.show_solution = False
        self.category_filter = category_filter or "All"
        self.filtered_indices = list(range(len(SCENARIOS)))

    def run(self):
        curses.curs_set(1)
        self.stdscr.keypad(True)
        init_colors()

        self.apply_category_filter()
        self.load_scenario()

        while True:
            self.draw()
            try:
                ch = self.stdscr.getch()
            except KeyboardInterrupt:
                break

            if ch == curses.KEY_RESIZE:
                continue
            elif ch == ord('q') or ch == 27:
                break
            elif ch in (curses.KEY_F1, ord('\t')): # Hint
                hints = SCENARIOS[self.filtered_indices[self.current_idx]].get("hints", [])
                if hints:
                    self.hint_level = (self.hint_level + 1) % (len(hints) + 1)
            elif ch == curses.KEY_F2: # Solution
                self.show_solution = not self.show_solution
                if self.show_solution:
                    self.streak = 0
            elif ch == curses.KEY_F3 or ch == 14: # F3 or Ctrl+N -> next
                self.next_scenario()
            elif ch == curses.KEY_F4: # F4 -> reset
                self.load_scenario()
                self.feedback = "Scenario reinitialized."
                self.feedback_success = False
                self.feedback_output = ""
            elif ch == curses.KEY_F5: # F5 -> category cycle
                self.cycle_category()
            elif ch == curses.KEY_F6: # F6 -> change session count
                self.prompt_session_count()
            elif ch in (curses.KEY_ENTER, 10, 13):
                self.submit_command()
            elif ch in (curses.KEY_BACKSPACE, 127, 8):
                if self.cursor_pos > 0:
                    self.user_cmd = self.user_cmd[:self.cursor_pos - 1] + self.user_cmd[self.cursor_pos:]
                    self.cursor_pos -= 1
            elif ch == curses.KEY_DC:
                if self.cursor_pos < len(self.user_cmd):
                    self.user_cmd = self.user_cmd[:self.cursor_pos] + self.user_cmd[self.cursor_pos + 1:]
            elif ch == curses.KEY_LEFT:
                if self.cursor_pos > 0:
                    self.cursor_pos -= 1
            elif ch == curses.KEY_RIGHT:
                if self.cursor_pos < len(self.user_cmd):
                    self.cursor_pos += 1
            elif ch == curses.KEY_HOME or ch == 1: # Ctrl+A
                self.cursor_pos = 0
            elif ch == curses.KEY_END or ch == 5: # Ctrl+E
                self.cursor_pos = len(self.user_cmd)
            elif ch == 21: # Ctrl+U
                self.user_cmd = ""
                self.cursor_pos = 0
            elif ch == curses.KEY_UP:
                if self.history:
                    if self.history_idx > 0:
                        self.history_idx -= 1
                    self.user_cmd = self.history[self.history_idx]
                    self.cursor_pos = len(self.user_cmd)
            elif ch == curses.KEY_DOWN:
                if self.history and self.history_idx < len(self.history) - 1:
                    self.history_idx += 1
                    self.user_cmd = self.history[self.history_idx]
                    self.cursor_pos = len(self.user_cmd)
                elif self.history_idx >= len(self.history) - 1:
                    self.history_idx = len(self.history)
                    self.user_cmd = ""
                    self.cursor_pos = 0
            elif 32 <= ch <= 126:
                self.user_cmd = self.user_cmd[:self.cursor_pos] + chr(ch) + self.user_cmd[self.cursor_pos:]
                self.cursor_pos += 1

        self.engine.cleanup()

    def prompt_session_count(self):
        max_y, max_x = self.stdscr.getmaxyx()
        self.stdscr.attron(curses.color_pair(COLOR_HEADER) | curses.A_BOLD)
        prompt_bar = " Set Session Question Count (e.g. 5, 10, 20, 50, or 'all'): "
        self.stdscr.addstr(max_y - 1, 0, prompt_bar.ljust(max_x - 1)[:max_x - 1])
        self.stdscr.attroff(curses.color_pair(COLOR_HEADER) | curses.A_BOLD)
        curses.echo()
        curses.curs_set(1)
        try:
            val = self.stdscr.getstr(max_y - 1, len(prompt_bar), 10).decode('utf-8', errors='ignore').strip()
            if val.lower() in ("all", "0", ""):
                self.session_count = None
                self.feedback = "Session target set to: All Questions."
            elif val.isdigit() and int(val) > 0:
                self.session_count = int(val)
                self.session_completed = 0
                self.feedback = f"Session target set to {self.session_count} questions."
        except Exception:
            pass
        finally:
            curses.noecho()
            curses.curs_set(1)

    def show_completion_modal(self):
        max_y, max_x = self.stdscr.getmaxyx()
        self.stdscr.clear()
        win_w = min(68, max_x - 4)
        start_x = (max_x - win_w) // 2
        curr_y = max(2, (max_y - 14) // 2)

        border = "═" * (win_w - 2)
        self.stdscr.attron(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)
        self.stdscr.addstr(curr_y, start_x, f"╔{border}╗")
        curr_y += 1
        title = "★ SESSION COMPLETED! ★"
        self.stdscr.addstr(curr_y, start_x, f"║{title.center(win_w - 2)}║")
        curr_y += 1
        self.stdscr.addstr(curr_y, start_x, f"╠{border}╣")
        curr_y += 1
        self.stdscr.attroff(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)

        lines = [
            f"You finished all {self.session_count} planned practice questions!",
            "",
            f"  Final Score: {self.score}",
            f"  Pass Accuracy: {int((self.successes / self.attempts * 100)) if self.attempts > 0 else 100}%",
            f"  Best Streak: {self.max_streak}",
            "",
            "Press [Enter] to start another session with same count,",
            "Press [C] to change question count, or [Q] to exit."
        ]
        for l in lines:
            self.stdscr.addstr(curr_y, start_x, f"║ {l.ljust(win_w - 4)} ║")
            curr_y += 1

        self.stdscr.attron(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)
        self.stdscr.addstr(curr_y, start_x, f"╚{border}╝")
        self.stdscr.attroff(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)
        self.stdscr.refresh()

        while True:
            k = self.stdscr.getch()
            if k in (ord('q'), 27):
                return False
            elif k in (curses.KEY_ENTER, 10, 13):
                self.session_completed = 0
                return True
            elif k in (ord('c'), ord('C')):
                self.prompt_session_count()
                return True

    def cycle_category(self):
        cats = ["All", "Service Management", "Firewall & Network", "Grep & Regex", "Sed Stream Editing", "Awk Processing", "Find & Permissions", "Pipes & Redirections"]
        cur_idx = cats.index(self.category_filter) if self.category_filter in cats else 0
        self.category_filter = cats[(cur_idx + 1) % len(cats)]
        self.apply_category_filter()
        self.current_idx = 0
        self.load_scenario()

    def apply_category_filter(self):
        if self.category_filter == "All":
            self.filtered_indices = list(range(len(SCENARIOS)))
        else:
            self.filtered_indices = [
                i for i, s in enumerate(SCENARIOS)
                if s["category"] == self.category_filter
            ]
        if not self.filtered_indices:
            self.filtered_indices = [0]

    def load_scenario(self):
        self.user_cmd = ""
        self.cursor_pos = 0
        self.feedback = None
        self.feedback_success = False
        self.feedback_output = ""
        self.hint_level = 0
        self.show_solution = False
        idx = self.filtered_indices[self.current_idx]
        self.engine.setup_scenario(SCENARIOS[idx])

    def next_scenario(self):
        self.current_idx = (self.current_idx + 1) % len(self.filtered_indices)
        self.load_scenario()

    def submit_command(self):
        if not self.user_cmd.strip():
            return
        self.history.append(self.user_cmd)
        self.history_idx = len(self.history)
        self.attempts += 1

        scenario = SCENARIOS[self.filtered_indices[self.current_idx]]
        success, output, msg = self.engine.execute(self.user_cmd, scenario)

        self.feedback_success = success
        self.feedback_output = output
        self.feedback = msg

        if success:
            self.successes += 1
            self.session_completed += 1
            self.streak += 1
            if self.streak > self.max_streak:
                self.max_streak = self.streak
            self.score += 100 + (self.streak * 25)
            if self.session_count and self.session_completed >= self.session_count:
                if not self.show_completion_modal():
                    sys.exit(0)
        else:
            self.streak = 0

    def draw(self):
        self.stdscr.erase()
        max_y, max_x = self.stdscr.getmaxyx()
        if max_y < 22 or max_x < 70:
            self.stdscr.addstr(0, 0, "Terminal window too small! Please resize to at least 80x24.")
            self.stdscr.refresh()
            return

        header_text = f" shell-drill [RHEL 9 MUSCLE-MEMORY TRAINER] | Cat: {self.category_filter} "
        self.stdscr.attron(curses.color_pair(COLOR_HEADER) | curses.A_BOLD)
        self.stdscr.addstr(0, 0, header_text.ljust(max_x - 1)[:max_x - 1])
        self.stdscr.attroff(curses.color_pair(COLOR_HEADER) | curses.A_BOLD)

        acc = int((self.successes / self.attempts * 100)) if self.attempts > 0 else 100
        sess_str = f"SESSION: {self.session_completed}/{self.session_count}" if self.session_count else "SESSION: All"
        stats = f"  STREAK: {self.streak} (Max: {self.max_streak}) | ACCURACY: {acc}% | SCORE: {self.score} | {sess_str} | DRILL {self.current_idx + 1}/{len(self.filtered_indices)}  "
        self.stdscr.attron(curses.color_pair(COLOR_WARN) | curses.A_BOLD)
        self.stdscr.addstr(1, 0, stats.center(max_x - 1)[:max_x - 1])
        self.stdscr.attroff(curses.color_pair(COLOR_WARN) | curses.A_BOLD)

        scenario = SCENARIOS[self.filtered_indices[self.current_idx]]

        curr_y = 3
        self.stdscr.attron(curses.color_pair(COLOR_CYAN) | curses.A_BOLD)
        self.stdscr.addstr(curr_y, 2, f"┌─── [{scenario['category']}] : {scenario['title']} ({scenario['difficulty']}) ".ljust(max_x - 4, "─") + "┐")
        self.stdscr.attroff(curses.color_pair(COLOR_CYAN) | curses.A_BOLD)
        curr_y += 1

        desc_lines = textwrap.wrap(scenario["description"], width=max_x - 8)
        for line in desc_lines:
            self.stdscr.addstr(curr_y, 4, f"│ {line}".ljust(max_x - 3) + "│")
            curr_y += 1

        self.stdscr.addstr(curr_y, 4, f"│ Target: {scenario['objective']}".ljust(max_x - 3) + "│")
        curr_y += 1

        files = list(scenario.get("setup_files", {}).keys())
        if files:
            files_str = f"│ Scratch Files: {', '.join(files)}"
            self.stdscr.addstr(curr_y, 4, files_str.ljust(max_x - 3) + "│")
            curr_y += 1

        self.stdscr.attron(curses.color_pair(COLOR_CYAN) | curses.A_BOLD)
        self.stdscr.addstr(curr_y, 2, "└" + ("─" * (max_x - 5)) + "┘")
        self.stdscr.attroff(curses.color_pair(COLOR_CYAN) | curses.A_BOLD)
        curr_y += 2

        hints = scenario.get("hints", [])
        if self.hint_level > 0 and hints:
            h_idx = min(self.hint_level - 1, len(hints) - 1)
            hint_txt = f" HINT [{self.hint_level}/{len(hints)}]: {hints[h_idx]} "
            self.stdscr.attron(curses.color_pair(COLOR_WARN))
            self.stdscr.addstr(curr_y, 4, hint_txt[:max_x - 6])
            self.stdscr.attroff(curses.color_pair(COLOR_WARN))
            curr_y += 1

        if self.show_solution:
            sol_txt = f" SOLUTION: {scenario['solution']} "
            self.stdscr.attron(curses.color_pair(COLOR_RED_HAT) | curses.A_BOLD)
            self.stdscr.addstr(curr_y, 4, sol_txt[:max_x - 6])
            self.stdscr.attroff(curses.color_pair(COLOR_RED_HAT) | curses.A_BOLD)
            curr_y += 1

        prompt_label = "[root@rhel9-snapshot scratch]$ "
        self.stdscr.attron(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)
        self.stdscr.addstr(curr_y, 2, prompt_label)
        self.stdscr.attroff(curses.color_pair(COLOR_SUCCESS) | curses.A_BOLD)

        self.stdscr.addstr(curr_y, 2 + len(prompt_label), self.user_cmd[:max_x - len(prompt_label) - 5])
        input_cursor_x = 2 + len(prompt_label) + self.cursor_pos
        input_cursor_y = curr_y
        curr_y += 2

        if self.feedback:
            status_tag = "[PASS ✓]" if self.feedback_success else "[FAIL ✗]"
            color = COLOR_SUCCESS if self.feedback_success else COLOR_ERROR
            self.stdscr.attron(curses.color_pair(color) | curses.A_BOLD)
            self.stdscr.addstr(curr_y, 2, f"{status_tag} {self.feedback}")
            self.stdscr.attroff(curses.color_pair(color) | curses.A_BOLD)
            curr_y += 1

            if self.feedback_output:
                self.stdscr.addstr(curr_y, 2, "┌── Output " + ("─" * (max_x - 14)) + "┐")
                curr_y += 1
                lines = self.feedback_output.splitlines()[:max(3, max_y - curr_y - 4)]
                for l in lines:
                    if curr_y < max_y - 3:
                        self.stdscr.addstr(curr_y, 4, l[:max_x - 6])
                        curr_y += 1
                if curr_y < max_y - 3:
                    self.stdscr.addstr(curr_y, 2, "└" + ("─" * (max_x - 4)) + "┘")
                    curr_y += 1

        hotkeys = " [Enter] Run  [Tab/F1] Hint  [F2] Solution  [F3] Skip  [F4] Reset  [F5] Cat  [F6] Count  [q] Quit "
        bottom_y = max_y - 1
        self.stdscr.attron(curses.color_pair(COLOR_HEADER))
        self.stdscr.addstr(bottom_y, 0, hotkeys.center(max_x - 1)[:max_x - 1])
        self.stdscr.attroff(curses.color_pair(COLOR_HEADER))

        try:
            self.stdscr.move(input_cursor_y, min(max_x - 2, input_cursor_x))
        except Exception:
            pass

        self.stdscr.refresh()


# ----------------------------------------------------------------------
# Library Import & Export Helpers
# ----------------------------------------------------------------------
def load_external_library(file_path):
    global SCENARIOS
    if not os.path.exists(file_path):
        print(f"Error: Library file not found: {file_path}")
        sys.exit(1)
    try:
        with open(file_path, "r", encoding="utf-8") as f:
            data = json.load(f)
            if isinstance(data, list):
                SCENARIOS = data
                print(f"[✓] Successfully loaded {len(SCENARIOS)} scenarios from: {file_path}")
            else:
                print(f"Error: JSON file must contain a list of scenarios.")
                sys.exit(1)
    except Exception as e:
        print(f"Error parsing library file: {e}")
        sys.exit(1)

def export_library(file_path):
    try:
        serializable = []
        for s in SCENARIOS:
            clean = dict(s)
            serializable.append(clean)
        with open(file_path, "w", encoding="utf-8") as f:
            json.dump(serializable, f, indent=2)
        print(f"[✓] Exported {len(serializable)} scenarios to {file_path}")
        sys.exit(0)
    except Exception as e:
        print(f"Error exporting library: {e}")
        sys.exit(1)


# ----------------------------------------------------------------------
# Command Line Arguments & Entry Point
# ----------------------------------------------------------------------
def parse_arguments():
    parser = argparse.ArgumentParser(
        prog="shell-drill",
        description="RHEL 8/9 Terminal Muscle-Memory Trainer for SysAdmins (Curses TUI)"
    )
    parser.add_argument(
        "--count", "-n",
        type=int,
        help="Number of questions to practice during this session (e.g. 5, 10, 25, 50)",
        default=None
    )
    parser.add_argument(
        "--category", "-c",
        help="Start with a specific category filter (e.g. 'Service Management', 'Firewall & Network')",
        default=None
    )
    parser.add_argument(
        "--library", "-L",
        help="Path to an external JSON scenarios library file to import",
        default=None
    )
    parser.add_argument(
        "--export-library",
        help="Export all built-in scenarios to a JSON library file and exit",
        default=None
    )
    parser.add_argument(
        "--list", "-l",
        action="store_true",
        help="List all available scenarios and exit"
    )
    return parser.parse_args()

def main():
    args = parse_arguments()

    if args.export_library:
        export_library(args.export_library)

    if args.library:
        load_external_library(args.library)

    if args.list:
        print("\n=== shell-drill RHEL 8/9 Scenario Library ===")
        print(f"{'#':<4} {'Category':<22} | {'Title':<48} ({'Difficulty'})")
        print("-" * 85)
        for idx, sc in enumerate(SCENARIOS, 1):
            print(f"[{idx:02d}] {sc['category']:<22} | {sc['title']:<48} ({sc['difficulty']})")
        print(f"\nTotal: {len(SCENARIOS)} scenarios available for disposable RHEL snapshot practice.\n")
        sys.exit(0)

    if not sys.stdout.isatty():
        print("Error: shell-drill requires an interactive TTY/terminal session (SSH or console).")
        print("Usage: python3 shell_drill.py")
        sys.exit(1)

    curses.wrapper(lambda scr: ShellDrillTUI(scr, category_filter=args.category, session_count=args.count).run())

if __name__ == "__main__":
    main()
