# Thunderbird command line

## Start

```sh
# start profile manager
thunderbird -P

# start with selected profile by name
thunderbird -P "<PROFILE-NAME>"

# start with selected profile by path (relative or absolute)
thunderbird -profile "<PROFILE-PATH>"

# allows multiple copies of application to be open at a time
thunderbird -no-remote ...

# start at offline mode by default
thunderbird -offline ...
```

# New gmail account creation

In OFFLINE mode:

- rename newly create SMTP server record to account email address;
- 'Synchronization & Storage -> Keep messages for this account on this computer' - uncheck;
- 'Server Settings -> Local directory' - rename directory to account email;
- restart;
- 'Synchronization & Storage -> Advanced' - uncheck '\[Gmail]/All Mail', '\[Gmail]/Spam', '\[Gmail]/Trash' to prevent downloading messages from this folders for offline use;
- 'Junk Settings -> Move new junk messages to' - select '\[Gmail]/Spam' to prevent new 'Junk' folder creation in gmail;
- 'Copies & Folders -> Place a copy in -> Other' - select '\[Gmail]/Sent Mail';
- 'Copies & Folders -> Keep message drafts in -> Other' - select '\[Gmail]/Drafts';

# Apply styles

<https://support.mozilla.org/en-US/questions/1198055>

Create a folder in your Firefox profile named chrome

In this folder, create a text file named userChrome.css

In that file place the following code:

```css
/*
 * Do not remove the @namespace line -- it's required for correct functioning
 */
@namespace url("http://www.mozilla.org/keymaster/gatekeeper/there.is.only.xul"); /* set default namespace to XUL */

/*
 * Make all the default font sizes 9 pt:
 */
* {
    font-size: 9pt !important;
}

/*
 * Make all the default font sizes 9 pt:
 */
* {
    font-size: 9pt !important;
    font-family: Arial !important;
}
```

Obviously, edit that `font-size: 9pt !important;` to a value that suits you. I generally use the same point size as was selected in the OS's desktop settings. You can also set the font face here, e.g.

A similar file named userContent.css in the same chrome folder can be used to set the size of the content of webpages (and of email if used with Thunderbird.)
