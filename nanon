#!/usr/bin/python

# nanon
# Copyright (C) 2008 Telmo Menezes.
# telmo (at) telmomenezes.com
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the version 2 of the GNU General Public License 
# as published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA

import sys
import glob
from HTMLParser import HTMLParser 
import htmlentitydefs
from getopt import *
import os

version = "0.1"

warnings = 0

class NMLParser(HTMLParser):
    def reset(self):
        self.output = ""
        self.vars = []
        self.file_list = []
        self.dir_path = ""
        HTMLParser.reset(self)
    def handle_starttag(self, tag, attrs):
        attrs_string = "".join([' %s="%s"' % (key, value) for key, value in attrs])

        file_name = ""
        if tag == "nanon":
            for key, value in attrs:
                if key == "file":
                    file_name = value
            if file_name not in self.file_list:
                try:
                    file = open(os.path.join(self.dir_path, file_name))
                    parser = NMLParser()
                    parser.vars = attrs
                    parser.file_list = self.file_list
                    parser.file_list.append(file_name)
                    parser.feed(file.read())
                    file.close()
                    self.output += parser.output
                except IOError:
                    global warnings
                    print "warning: could not open '" + file_name + "'"
                    warnings += 1
        elif tag == "nanonv":
            varName = ""
            for key, value in attrs:
                if key == "var":
                    varName = value
            for key, value in self.vars:
                if key == varName:
                    self.output += value
        else:
            self.output += "<" + tag + attrs_string + ">"
    def handle_endtag(self, tag):
        if tag == "nanon":
            pass
        elif tag == "nanonv":
            pass
        else:
            self.output += "</" + tag + ">"
    def handle_data(self, text):
        self.output += text
    def handle_charref(self, ref):
        self.output += "&#" + ref
    def handle_entityref(self, ref):
        self.output += "&" + ref
        if htmlentitydefs.entitydefs.has_key(ref):
            self.output += ";"
    def handle_comment(self, text):
        self.output += "<!--" + text + "-->"
    def handle_decl(self, text): 
        self.output += "<!" + text + ">"
    def handle_pi(self, text):
        self.output += "<?" + text + ">"

def process_file(file_name, dir_path):
    if file_name[-4:] != ".nml":
        return 0

    file = open(os.path.join(dir_path, file_name), "rb")
    parser = NMLParser()
    parser.dir_path = dir_path
    parser.feed(file.read())
    file.close()
    out_file_name = file_name[0:-4] + ".html"
    file = open(os.path.join(dir_path, out_file_name), "w")
    file.write(parser.output)
    file.close()
    print file_name + " processed, " + out_file_name + " generated."
    return 1

def help():
    print "usage: " + sys.argv[0] + " [-h -r] source_file ..." 

def show_version():
    print "nanon " + version + " by Telmo Menezes"

try:
    opts, args = getopt(sys.argv[1:], ":hrv")
except GetoptError:
    help()
    exit()

if ('-v', '') in opts:
    show_version()
    exit()

if len(args) == 0:
    help()
    exit()

recurse = False
if ('-r', '') in opts:
    recurse = True

files_processed = 0

for arg in args:
    file_list = glob.glob(arg)
    for file_path in file_list:
        dir_path, file_name = os.path.split(arg)
        files_processed += process_file(file_name, dir_path)
        if recurse:
            for root, dirs, files in os.walk(file_name):
                for name in files:
                     files_processed += process_file(name, root)

print "" + str(files_processed) + " file(s) processed, " + str(warnings) + " warning(s)."
print "done!"
