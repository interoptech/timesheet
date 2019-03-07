# Installation

Timesheet can be installed using pre-build docker images or as a binary.

## Requirements

Application can run in [Docker containers](https://hub.docker.com/search/?type=edition&offering=community). Server image size 24.9 MB, DB image size 312 MB.

If you are running it directly, supported is:
- Linux, Windows or MacOS
- PostgreSQL DB

# Quick start

1. Update Application settings, Vacation settings and Warning limits to meet your needs. See Configuration chapter.
2. Use Backup & Restore to export demo data, update csv files and import back your projects, rates, consultants and holidays. See Backup & Restore chapter.

Enjoy. You are good to go ...

# Configuration

Below is default and commented `timesheet.yaml` configuration file shipped with a product. Selected variables can be changed on Administration page.

![Administration](./images/administration.png?raw=true "Administration")

```
### Default configuration file

######################
# Reporting settings #

# Set your timezone. Supported timezones: https://github.com/moment/moment-timezone/blob/develop/data/packed/latest.json
timeZone: "Europe/Prague"

dailyWorkingHours: 8 # Used to calculate weekly and monthly expected working hours, can be changed in UI
dailyWorkingHoursMin: 8 # Used to highlight if reported less, can be changed in UI
dailyWorkingHoursMax: 12 # Used to highlight if reported more, can be changed in UI

# Rate used for vacations
vacation: "Vacation"
yearlyVacationDays: 20 # Used to calculate weekly and monthly expected working hours, can be changed in UI

# Rate for additonal vacations. If not used, leave blank "" and set yearlyPersonalDays: 0, can be changed in UI
vacationPersonal: "Vacation Personal"
yearlyPersonalDays: 3 # Used to calculate weekly and monthly expected working hours, can be changed in UI

# Rate used for additonal vacation intended for sick day. If not used, leave blank "" and set yearlySickDays: 0, can be changed in UI
vacationSick: "Vacation Sick"
yearlySickDays: 2 # Used to calculate weekly and monthly expected working hours, can be changed in UI

# Categorize all rates into one of these types used on Reported Overview page
isWorking: "work" # when consultant works, can be changed in UI
isNonWorking: "not-work" # when consultant dows not work, examples: vacation, sick, personal day, public holiday, vacation, unpaid leave, ..., can be changed in UI

########################
# Application settings #
url: "" # URL on which application is running
PORT: "3000"     # port on which application is running

# DB type
dbType: "postgresql"

# Production URL - will be read from production environemnt config variable
# If set, Database settings section variables will be ignored
DATABASE_URL: ""

# Log folder, relative to timesheet folder
logFolder: "logs"

# # Folder for uploaded data, relative to timesheet folder
uploadFolder: "data/uploaded"
uploadFolderTemp: "data/uploaded/temp"

# csv data files which are loaded via command "timesheet db --load all"
data:
  consultants: "consultants_prod.csv"
  rates: "rates_prod.csv"
  projects: "projects_prod.csv"
  reportedRecords: "reported_records_prod.csv"
  holidays: "holidays_cz_2019.csv"

export:
  location: "data/exported" # select an empty and an existing folder

#####################
# Database settings #

# DB backup settings - backuped data can be imported directly by a command "timesheet db --load all"
backup:
  location: "data/backups" # select an empty and an existing folder relative to timesheet/data folder
  rotation: 14             # how many backups back will be kept
  interval: "daily"        # daily or weekly - how ofter the DB backup should be done

# DB credentials
# used for development and testing. Ignored if DATABASE_URL is set
postgresql:
# host: "db" #
  host: "127.0.0.1" #
  port: "5432"
  name: "timesheet"
  user: "timesheet"
  password: "timesheet"
  sslMode: "disable"
```

## Command Line Options

`.\timesheet.exe` or `.\timesheet.bin` or `.\timesheet.app`
```
Web based timesheet application with DB persistence.

Application reads DB and server configuration from timesheet.toml, loads default data if DB is empty and launch web GUI.

Usage:
  timesheet [command]

Available Commands:
  db          Initiate, load or backup DB. See timesheet help db
  help        Help about any command
  routes      Prints the list of all available routes
  server      Starts the server on URL and port defined in config.yaml

Flags:
      --config string   config file (default is ./timesheet.yaml)
  -h, --help            help for timesheet
  -v, --version         Prints application versions

Use "timesheet [command] --help" for more information about a command.
```

`.\timesheet.exe help db` or `.\timesheet.bin  help db` or `.\timesheet.app  help db`
```
Initiate, load, or backup DB.

Command first tests connection to DB. If succeeds it will initiate, load or backup db and exit.

Usage:
  timesheet db [flags]

Flags:
  -b, --backup        Backup all DB tables in the format used by db --load command
  -c, --clean         Drop and create all required DB tables
  -h, --help          help for db
  -l, --load string   Truncate DB table/tables and load initial data from files in folder ./data. Options:
                      all - load all tables
                      rates | consultants | projects | holidays | reported_records - load selected table

Global Flags:
      --config string   config file (default is ./timesheet.yaml)
```

# Backup & Restore

All data can be downloaded locally as a zip file including csv files.
Files can be modified and imported back. This action will replace all existing data.

![Backup & Restore](./images/backup_restore.png?raw=true "Backup & Restore")

Check log files using Administration / Logs. Error log should not contain any errors or wardnings.

## Description of Data Files

Use ISO format YYYY-MM-DD HH:MM:SS for all date fields.

### consultants.csv

Contains all consultants which can report hours.
```
created_at,name
"YYYY-MM-DD HH:MM:SS","consultant name"
"2019-01-01 00:00:00","Evan You"
```

### holidays.csv

Contains state holidays per selected year.
```
created_at,date,description
"YYYY-MM-DD HH:MM:SS","name of the holiday"
"2019-01-01 00:00:00","2019-01-01","New Year's Day"
```

### rates.csv

Contains rates which can be selected by any consultant.
Rates are categorized into working *work* and *non-working*. Category names can be changed but update also timesheet.yaml.
```
created_at,name,type
"YYYY-MM-DD HH:MM:SS","rate name","work or non-work"
"2019-01-14 00:00:00","On-site","work"
"2019-01-14 00:00:00","Vacation","not-work"
```

# projects.csv

Contains projects on which consultants can report the work and default project rate.
```
created_at,name,rate
"YYYY-MM-DD HH:MM:SS","project name","rate name"
"2019-01-14 00:00:00","Vue","Off-site"
```

# reported_records.csv

Contains all reported hours. Hours (N.N) can be decimal number between 0 and 24. Project, rate and consultant are names of existing records from corresponding csv files.
```
created_at,date,hours,project,description,rate,consultant
YYYY-MM-DD HH:MM:SS,YYYY-MM-DD,N.N,project name,description of the work,rate name,project name
2019-01-10 10:00:00,2019-01-01,6,Vue,Updates of all Vue.js documentation examples using typescript,Off-site,Evan You
```

# Upgrade

Follow these steps to upgrade:
* Export your data as described in Backup & Restore
* Replace `timesheet` folder with a new version
* Import saved data as described in Backup & Restore


# License

Free for education and non-commertial usage. Pay for the commertial usage of the application to support further development and maintenance via
<a href="https://www.patreon.com/valasek">Patreon</a> or <a href="https://paypal.me/StanislavValasek">PayPal</a>.

My plan is to soon start working on a Pro Version of Timesheet for enterprise. Along with support, some of the planned features include:

* User management/login
* Permissions
* HTTPS
* Plugin for import/export 
* Reporting metrics
* Cloud version

If you or your organization would like to help beta test a Pro version of Timesheet, please get in touch with me:

    Twitter: @valasek
    Email: valasek at gmail.com

# Release Notes

## Version 1.1.1
Released on March 7, 2019

### Usability

* Previous weeks are loaded laster
* Weeks and consultants can be changed directly on the Overview page
* Removed footer, main toolbar takes less space
* Disable all fields in previous weeks unless editing is enabled
* Warn if entered date is not from this week
* Validation on reported hours, allowed number: 0 - 24
* Validate maximum reported daily hours during record duplication
* Validations on entering hours and days

### Fixes

* Backup archive contains exported csv files without subfolders

## Version 1.0.0
Released on December, 2018

The first publicly released version