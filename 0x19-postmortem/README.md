Postmortem
Issue Summary
The companmy's Server go down for 10 minutes on December 4th 2022. The incident occurred between 6:30pm to 6:40pm WAT. GET requests resulted in 500 internal server error and all users were affected. After a minor update to app, a script asccidentally stopped the web server nginx from running.

Debugging Process
Bug debugger Bamidele (Lexxyla... as in my actual initials... made that up on the spot, pretty good, huh?) encountered the issue upon opening the project and being, well, instructed to address it, roughly 19:20 PST. He promptly proceeded to undergo solving the problem.

Checked running processes using ps aux. Two apache2 processes - root and www-data - were properly running.

Looked in the sites-available folder of the /etc/apache2/ directory. Determined that the web server was serving content located in /var/www/html/.

In one terminal, ran strace on the PID of the root Apache process. In another, curled the server. Expected great things... only to be disappointed. strace gave no useful information.

Repeated step 3, except on the PID of the www-data process. Kept expectations lower this time... but was rewarded! strace revelead an -1 ENOENT (No such file or directory) error occurring upon an attempt to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

Looked through files in the /var/www/html/ directory one-by-one, using Vim pattern matching to try and locate the erroneous .phpp file extension. Located it in the wp-settings.php file. (Line 137, require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).

Removed the trailing p from the line.

Tested another curl on the server. 200 A-ok!

Wrote a Puppet manifest to automate fixing of the error.

Summation
In short, a typo. Gotta love'em. In full, the WordPress app was encountering a critical error in wp-settings.php when tyring to load the file class-wp-locale.phpp. The correct file name, located in the wp-content directory of the application folder, was class-wp-locale.php.

Patch involved a simple fix on the typo, removing the trailing p.

Corrective and Preventative Measures
The develops team had a meeting and agreed that after each deployment, the status of the server should be checked to prevent the same event in the nearest future.
