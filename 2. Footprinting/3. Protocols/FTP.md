[Commands](https://www.smartfile.com/blog/the-ultimate-ftp-commands-list/)
Clear Text Protocols
Anonymous login?
TFTP
	Trivial File Transfer Protocol
	Simpler, yet does not provide user authentication
	Uses UDP
vsFTPd
	Configuration found in /etc/vsftpd.conf
		[Possible settings](http://vsftpd.beasts.org/vsftpd_conf.html)
	/etc/ftpusers
		Deny certain users.
	Dangerous Settings
		anonymous_enable=YES
			Allowing anonymous login?
		anon_upload_enable=YES
			Allowing anonymous to upload files?
		anon_mkdir_write_enable=YES
			Allowing anonymous to create new directories?
		no_anon_password=YES
			Do not ask anonymous for password?
		anon_root=/home/username/ftp
			Directory for anonymous.
		write_enable=YES
			Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?
			
Debug and Trace command?
Recursive Listing 
	`ls -R`
Download all files
	May cause alarm bells?
		`wget -m --no-passive ftp://anonymous:anonymous@<ip>`
	tree . 
NMAP
	ftp-anon
	--script-trace