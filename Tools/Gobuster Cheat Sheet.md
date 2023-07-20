Common Command line options

    -fw – force processing of a domain with wildcard results.
    -np – hide the progress output.
    -m  – which mode to use, either dir or dns (default: dir).
    -q – disables banner/underline output.
    -t
     – number of threads to run (default: 10).
    -u  – full URL (including scheme), or base domain name.
    -v – verbose output (show all results).
    -w  – path to the wordlist used for brute forcing (use – for stdin).

Command line options for dns mode

    -cn – show CNAME records (cannot be used with ‘-i’ option).
    -i – show all IP addresses for the result.

Command line options for dir mode

    -a <user agent string> – specify a user agent string to send in the request header.
    -c <http cookies> – use this to specify any cookies that you might need (simulating auth).
    -e – specify extended mode that renders the full URL.
    -f – append / for directory brute forces.
    -k – Skip verification of SSL certificates.
    -l – show the length of the response.
    -n – “no status” mode, disables the output of the result’s status code.
    -o <file> – specify a file name to write the output to.
    -p <proxy url> – specify a proxy to use for all requests (scheme much match the URL scheme).
    -r – follow redirects.
    -s <status codes> – comma-separated set of the list of status codes to be deemed a “positive” (default: 200,204,301,302,307).
    -x <extensions> – list of extensions to check for, if any.
    -P <password> – HTTP Authorization password (Basic Auth only, prompted if missing).
    -U <username> – HTTP Authorization username (Basic Auth only).
    -to <timeout> – HTTP timeout. Examples: 10s, 100ms, 1m (default: 10s).

