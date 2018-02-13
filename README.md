INFO:

Refers to document CIS_Apple_OSX_10.12_Benchmark_v1.0.0.pdf, available at https://benchmarks.cisecurity.org

USAGE:

Create a single Jamf Policy using all three scripts.

	1_Set_Organization_Priorities - Script Priority: Before
	2_Security_Audit_Compliance - Script Priority: Before
	3_Security_Remediation - Script Priority: Before
	2_Security_Audit_Compliance - Script Priority: After
	
Upload the 3 Configuration Profiles to your Jamf Pro server

# 1_Set_Organization_Priorities

Policy: Some recurring trigger to track compliance over time.

Admins set organizational compliance for each listed item, which gets written to plist. The values default to "true," meaning if an organization wishes to disregard a given item they must set the value to false by changing the associated comment:

OrgScore1_1="true" or OrgScore1_1="false"

The script writes to /Library/Application Support/SecurityScoring/org_security_score.plist by default.

NOTES: 

* Item "1.1 Verify all Apple provided software is current" is disabled by default.
* Item "2.1.2 Turn off Bluetooth "Discoverable" mode when not pairing devices - not applicable to 10.9 and higher."
	Starting with OS X (10.9) Bluetooth is only set to Discoverable when the Bluetooth System Preference is selected. 
	To ensure that the computer is not Discoverable do not leave that preference open.
* Item "2.3.3 Verify Display Sleep is set to a value larger than the Screen Saver (Not Scored)"
	The rationale in the CIS Benchmark for this is incorrect. The computer will lock if the 
	display sleeps before the Screen Saver activates
* Item "2.6.6 Enable Location Services (Not Scored)" is disabled by default.
	As of macOS 10.12.6, Location Services cannot be enabled/monitored programmatically.
	It is considered user opt in.
* Item "2.6.7 Monitor Location Services Access (Not Scored)" is disabled by default.
	As of macOS 10.12.6, Location Services cannot be enabled/monitored programmatically.
	It is considered user opt in.
* Item "2.8.1 Time Machine Auto-Backup " is disabled by default.
	Time Machine is typically not used as an Enterprise backup solution
* Item "2.8.2 Time Machine Volumes Are Encrypted (Not Scored)" is disabled by default.
	Time Machine is typically not used as an Enterprise backup solution
* Item "2.12 Securely delete files as needed (Not Scored)" is disabled by default.
	With the wider use of FileVault and other encryption methods and the growing use of Solid State Drives
	the requirements have changed and the "Secure Empty Trash" capability has been removed from the GUI.
* Item "3.4 Enable remote logging for Desktops on trusted networks (Not Scored)" is disabled by default.
	The built-in syslog capability in OS X runs over UDP without encryption. Broadcasting log unencrypted over 
	the internet is not a good idea. While syslog may be acceptable on some internal trusted networks it is not a 
	solution for mobile devices that hop between networks.
* Item "4.3 Create network specific locations (Not Scored)" is disabled by default.
* Item "5.4 Automatically lock the login keychain for inactivity" is disabled by default.
* Item "5.5 Ensure login keychain is locked when the computer sleeps" is disabled by default.
* Item "5.6 Enable OCSP and CRL certificate checking" is disabled by default.
* Item "5.14 Do not enter a password-related hint (Not Scored)" is disabled by default.
	Not needed if 6.1.2 Disable "Show password hints" is enforced.
* Item "5.16 Secure individual keychains and items (Not Scored)" is disabled by default.
* Item "5.17 Create specialized keychains for different purposes (Not Scored)" is disabled by default.
* Item "5.19 Install an approved tokend for smartcard authentication" is disabled by default.
	This is superseded by the macos 10.12.x built in SmartCardServices and CryptoTokenKit. 
* Item "6.4 Safari disable Internet Plugins for global use (Not Scored)" is disabled by default.
* Item "6.5 Use parental controls for systems that are not centrally managed (Not Scored)" is disabled by default.

# 2_Security_Audit_Compliance

Policy: Some recurring trigger to track compliance over time. Run this before and after 3_Security_Remediation to audit the Remediation

Reads the plist at /Library/Application Support/SecurityScoring/org_security_score.plist. For items prioritized (listed as "true,") the script queries against the current computer/user environment to determine compliance against each item.

Non-compliant items are recorded at /Library/Application Support/SecurityScoring/org_audit

# 2.5_Audit_List Extension Attribute

Set as Data Type "String."

Reads contents of /Library/Application Support/SecurityScoring/org_audit file and records to Jamf Pro inventory record.

# 2.6_Audit_Count Extension Attribute

Set as Data Type "Integer." 

Reads contents of /Library/Application Support/SecurityScoring/org_audit file and records count of items to Jamf Pro inventory record. Usable with smart group logic (2.6_Audit_Count greater than 0) to immediately determine computers not in compliance.

# Configuration Profiles

CIS 10.12 LoginWindow Security remediates the following:

* 2.3.1 Set an inactivity interval of 20 minutes or less for the screen saver - LoginWindow payload > Options > Start screen saver after: 20 Minutes of Inactivity
* 2.3.2 Secure screen saver corners - Custom payload > com.apple.dock > wvous-tl-corner=0, wvous-br-corner=5, wvous-bl-corner=0, wvous-tr-corner=0
* 2.3.4 Set a screen corner to Start Screen Saver - Custom payload > com.apple.dock > wvous-tl-corner=0, wvous-br-corner=5, wvous-bl-corner=0, wvous-tr-corner=0
* 2.6.2 Enable Gatekeeper - Security and Privacy payload > General > Gatekeeper > Mac App Store and identified developers (selected)
* 2.6.3 Enable Firewall - Security and Privacy payload > Firewall > Enable Firewall (checked)
* 2.6.4 Enable Firewall Stealth Mode - Security and Privacy payload > Firewall > Enable stealth mode (checked)
* 2.6.5 Review Application Firewall Rules - Security and Privacy payload > Firewall > Control incoming connections for specific apps (selected)
* 2.7.1.02 Disable Apple ID setup during login - LoginWindow payload > Options >  Disable Apple ID setup during login (checked)
* 5.8 Disable automatic login - LoginWindow payload > Options > Disable automatic login (checked)
* 5.9 Require a password to wake the computer from sleep or screen saver - Security and Privacy payload > General > Require password * after sleep or screen saver begins (checked)
* 5.12 Create a custom message for the Login Screen - LoginWindow payload > Window > Banner (message)
* 5.15 Disable Fast User Switching - LoginWindow payload > Options > Enable Fast User Switching (unchecked)
* 6.1.1 Display login window as name and password - LoginWindow payload > Window > LOGIN PROMPT > Name and password text fields (selected)
* 6.1.2 Disable "Show password hints" - LoginWindow payload > Options > Show password hint when needed and available (checked - Yes this is backwards)
* 6.1.3 Disable guest account - LoginWindow payload > Options > Allow Guest User (unchecked)

CIS 10.12 Restrictions remediates the following:

* 2.7.1.02 Disable the iCloud system preference pane (Not Scored) - Restrictions payload > Preferences > disable selected items > iCloud
* 2.7.1.03 Disable the use of iCloud password for local accounts (Not Scored)  - Restrictions payload > Functionality > Allow use of iCloud password for local accounts (unchecked)
* 2.7.1.04 Disable iCloud Back to My Mac (Not Scored) - Restrictions payload > Functionality > Allow iCloud Back to My Mac (unchecked)
* 2.7.1.05 Disable iCloud Find My Mac (Not Scored) - Restrictions payload > Functionality > Allow iCloud Find My Mac (unchecked)
* 2.7.1.06 Disable iCloud Bookmarks (Not Scored) - Restrictions payload > Functionality > Allow iCloud Bookmarks (unchecked)
* 2.7.1.07 Disable iCloud Mail (Not Scored)  - Restrictions payload > Functionality > Allow iCloud Mail (unchecked)
* 2.7.1.08 Disable iCloud Calendar (Not Scored) - Restrictions payload > Functionality > Allow iCloud Calendar (unchecked)
* 2.7.1.09 Disable iCloud Reminders (Not Scored) - Restrictions payload > Functionality > Allow iCloud Reminders (unchecked)
* 2.7.1.10 Disable iCloud Contacts (Not Scored) - Restrictions payload > Functionality > Allow iCloud Contacts (unchecked)
* 2.7.1.11 Disable iCloud Notes (Not Scored) - Restrictions payload > Functionality > Allow iCloud Notes (unchecked)
* 2.7.1.12 Disable Content Caching (Not Scored) - Restrictions payload > Functionality > Allow Content Caching (unchecked)
* 2.7.2 Disable iCloud keychain (Not Scored) - Restrictions payload > Functionality > Allow iCloud Keychain (unchecked)
* 2.7.3 Disable iCloud Drive (Not Scored) - Restrictions payload > Functionality > Allow iCloud Drive (unchecked)
* 2.7.4 Disable iCloud Drive Document sync - Restrictions payload > Functionality > Allow iCloud Desktop & Documents (unchecked)
* 2.7.5 Disable iCloud Drive Desktop sync - Restrictions payload > Functionality > Allow iCloud Desktop & Documents (unchecked)

CIS 10.12 Custom Settings remediates the following:

* 1.2 Enable Auto Update - Custom payload > com.apple.SoftwareUpdate.plist > AutomaticCheckEnabled=true, AutomaticDownload=true
* 1.4 Enable system data files and security update installs - Custom payload > com.apple.SoftwareUpdate.plist > ConfigDataInstall=true, CriticalUpdateInstall=true
* 2.10 Enable Secure Keyboard Entry in terminal.app - Custom payload > com.apple.Terminal > SecureKeyboardEntry=true
* 4.1 Disable Bonjour advertising service - Custom payload > com.apple.mDNSResponder > NoMulticastAdvertisements=true
* 6.1.4 Disable Allow guests to connect to shared folders - Custom payload > com.apple.AppleFileServer guestAccess=false, com.apple.smb.server AllowGuestAccess=false
* 6.3 Disable the automatic run of safe files in Safari - Custom payload > com.apple.Safari > AutoOpenSafeDownloads=false

.mobileconfig files for all three profiles are provided to upload to your Jamf Pro server.  If you wish to disable remediation for any of the custom settings, the appropritiate plists have also been provided to edit then upload to Custom Settings Configuration Profile. 

# 3_Security_Remediation

Policy: Some recurring trigger to enforce compliance over time. Run 2_Security_Audit_Compliance after to audit the Remediation

Reads the plist at /Library/Application Support/SecurityScoring/org_security_score.plist. For items prioritized (listed as "true,") the script applies recommended remediation actions for the client/user.

SCORED CIS EXCEPTIONS:

- Does not implement pwpolicy commands (5.2.1 - 5.2.8)

- Audits but does not actively remediate (due to alternate profile/policy functionality within Jamf Pro):
* 2.4.4 Disable Printer Sharing
* 2.6.1 Enable FileVault
* 2.7.1 iCloud configuration (Check for iCloud accounts) (Not Scored)
* 2.11 Java 6 is not the default Java runtime
* 5.18 System Integrity Protection status

Remediated using configuration profiles

* 1.2 Enable Auto Update
* 1.4 Enable system data files and security update instal
* 2.3.1 Set an inactivity interval of 20 minutes or less for the screen saver 
* 2.3.2 Secure screen saver corners 
* 2.3.4 Set a screen corner to Start Screen Saver 
* 2.6.2 Enable Gatekeeper
* 2.6.3 Enable Firewall 
* 2.6.4 Enable Firewall Stealth Mode 
* 2.6.5 Review Application Firewall Rules 
* 2.7.1.01 Disable Apple ID setup during login (Not Scored)
* 2.7.1.02 Disable the iCloud system preference pane (Not Scored)
* 2.7.1.03 Disable the use of iCloud password for local accounts (Not Scored)
* 2.7.1.04 Disable iCloud Back to My Mac (Not Scored)
* 2.7.1.05 Disable iCloud Find My Mac (Not Scored)
* 2.7.1.06 Disable iCloud Bookmarks (Not Scored)
* 2.7.1.07 Disable iCloud Mail (Not Scored)
* 2.7.1.08 Disable iCloud Calendar (Not Scored)
* 2.7.1.09 Disable iCloud Reminders (Not Scored)
* 2.7.1.10 Disable iCloud Contacts (Not Scored)
* 2.7.1.11 Disable iCloud Notes (Not Scored)
* 2.7.1.12 Disable Content Caching (Not Scored)
* 2.7.2 iCloud keychain (Not Scored)
* 2.7.3 iCloud Drive (Not Scored)
* 2.7.4 iCloud Drive Document sync
* 2.7.5 iCloud Drive Desktop sync
* 2.10 Enable Secure Keyboard Entry in terminal.app 
* 4.1 Disable Bonjour advertising service 
* 5.8 Disable automatic login 
* 5.9 Require a password to wake the computer from sleep or screen saver
* 5.12 Create a custom message for the Login Screen
* 5.15 Disable Fast User Switching (Not Scored)
* 6.1.1 Display login window as name and password 
* 6.1.2 Disable "Show password hints" 
* 6.1.3 Disable guest account 
* 6.1.4 Disable "Allow guests to connect to shared folders" 
* 6.3 Disable the automatic run of safe files in Safari
