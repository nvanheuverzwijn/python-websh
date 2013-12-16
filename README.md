python-websh
============

A minimalist python web shell client 

When you find a vulnerable website where you can execute shell commands, the python web shell lets you connect to it and browse it as you would a regular directory. 

Example:
./websh http://www.example.com/shell.php?c=



# USAGE
	usage: websh [-h] [-e ENC] url

	Let the web be command-line.

	positional arguments:
	  url                Enter the full target URL where the shell/cmd can be
	                     executed.Example: www.example.com/shell.php
	
	optional arguments:
	  -h, --help         show this help message and exit
	  -e ENC, --enc ENC  Encoding/encryption to use. (Default: plain)


Possible encoding 
	ase64 or plain
