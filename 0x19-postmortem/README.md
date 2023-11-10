# Postmortem

About 06:07 West African Time (WAT), when ALX's System Engineering & DevOps project 0x19 was released, an outage happened on an isolated Ubuntu 14.04 container that was running an Apache web server. When an HTML file specifying a basic Holberton WordPress site was expected as response to GET queries on the server, `500 Internal Server Error` messages were sent.

## Debugging process

Mark Oladeinde discovered the problem when the project opened and was, uh, told to fix it at about 10:20 WAT. He immediately went about resolving the issue.

1. Utilizing `ps aux`, I verified the processes that were currently running. `root` and `www-data`, two instances of the `apache2` process, were operational.

2. Examined the `sites-available` subdirectory under the `/etc/apache2/` directory. found that content from `/var/www/html/` was being served by the web server.

3. Use the PID of the `root` Apache process to run `strace` in a single terminal. Curled the server in another. I was let down after having high expectations. `strace` provided no meaningful information.

4. Re-ran step 3, omitting the `www-data` process's PID. This time, I had fewer expectations, and I was not disappointed! An `-1 ENOENT (No such file or directory)` error was discovered by `strace` when trying to access the file `/var/www/html/wp-includes/class-wp-locale.phpp`.

5. Sifted through each file in the `/var/www/html/` directory one by one and used Vim pattern matching to look for the incorrect `.phpp` file extension. You may find it in the `wp-settings.php` file. (`require_once( ABSPATH. WPINC. '/class-wp-locale.php' );`, line 137).

6. Removed the line's trailing `p`.

7. Ran one more `curl` test on the server. 200 A-all right!

8. Created a Puppet manifest to automate the error's correction.

## Summation

Simply said, a typo. Must adore them. The WordPress application was unable to load the file `class-wp-locale.phpp` because of a serious error in `wp-settings.php`. The file `class-wp-locale.php` was the right name, and it was in the `wp-content` directory of the application folder.

Patch consisted of a straightforward typo correction—removing the trailing `p`.

## Prevention

An application fault, not a web server error, was the cause of this outage. Please bear the following in mind to avoid such interruptions in the future.

* Try it! Examine, test, test. Before launching, test the application. If the app had been tested, this issue would have surfaced and been resolved sooner.

* Tracking of status. To receive rapid notifications in the event of a website outage, enable an uptime monitoring service like [UptimeRobot](./https://uptimerobot.com/).

Naturally, though, since we're programmers and never make mistakes, it won't happen again! :wink:
