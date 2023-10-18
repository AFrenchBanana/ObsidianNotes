* Log analytics tool used to gather, analyse and visualise data. 
* Often used as SIEM
* Often used to house sensitive data.
* Not many vulnerabilities, so most focus on weak authentication. 
# Discovery and Enumeration 
## Discovery
* Often run as root or SYSTEM. 
* Rare to be seen externally facing. 
* Runs on port 8000 as default.
* Default creds `admin:changem`
	* Newer versions set credentials on installation.
	* Common weak passwords like `admin`,`welcome1 etc...
## Enumeration 
* The splunk enterprise trial converts to free after 60days which doesn't require auth.
* 