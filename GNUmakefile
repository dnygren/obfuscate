###############################################################################
# GNUmakefile : A Makefile to obfuscate shell scripts
#
# Created by Daniel C. Nygren
# Permanent Email: dan.nygren@gmail.com
#
# Copyright (c) 2001, 2020, by Daniel C. Nygren.
#
# BSD 0-clause license, "Zero Clause BSD", SPDX: 0BSD
#
#     This is a Makefile for use with shell scripts that you want to obscure
# the contents of. It first strips off blank lines, then those that have
# leading whitespace and lines that begin with a comment. It then inserts a
# line at the start of the script to turn off execution of tracing. The
# script is compressed, encrypted with the name of the target script as the
# key, and then a wrapper is placed around it to make it reverse the process
# when executed. The permissions are changed so anyone can execute it.
#     As the encryption key is the target script's name, if you want to
# password protect the file, simply rename the file. Restoring the filename to
# its original value will then allow the file to be executed.
#     Compressing the script obviously makes it more compact, but it also
# has the side effect of potentially making it more difficult to decipher the
# encryption. I say potentially because, if for instance you wanted to use as
# a compression engine the obsolete /usr/bin/compress, it automatically
# puts a three byte header of 0x1F 0x9D 0x90 at the start of the file. This
# could make it easier to decrypt if known. Since our goal is obfuscation, not
# strong encryption, it doesn't matter that much, but it is at least worth
# pointing this out.
#     The debug option creates a stripped version of the file for use in
# debugging. Errors that occur in the encrypted script report a line number.
# This line number is different that the one in the unstripped file, so a
# stripped yet unobfuscated version of the file is generated so that the
# offending line can be found.
#
# CALL SEQUENCE make [options] [target]
#
# EXAMPLE       make                 (Makes all targets)
#               make SOURCE=foo.sh   (Override file named in Makefile)
#               make debug           (Makes an unencrypted debug version)
#               make clean           (Deletes all targets)
#
# TARGET SYSTEM
#
# DEVELOPED ON Solaris 9, Linux
#
# CALLS        GNU sed, gzip, zcat, openssl, chmod, rm
#
# CALLED BY    make
#
# INPUTS       shell script, SOURCE, TARGET, COPYRIGHT
#
# OUTPUTS      wrapped encrypted script
#
# RETURNS      GNU sed returns 0 if OK, 1 if error in Script,
#
# ERRORS       make gives up on the current rule if there is an error
#
# WARNINGS     1) GNU make, and GNU sed are required to be named
#              make, and sed respectively, and in your path before
#              any other identically named executables.
#
###############################################################################

# An alternate source file name can be provided on the command line:
# make SOURCE=foo.sh
# Name of source shell script to be encrypted and wrapped
SOURCE := demo.sh

# Name of target script that is generated.
TARGET := obfuscated
# As the target name is the CRYPT key, long names are desirable, but not
# necessary.

# Copyright string
COPYRIGHT :=\# Copyright (c) 2020, by Daniel C. Nygren.

# ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
# ^^^^^^^^^^ Place code that may need modification above this point. ^^^^^^^^^^
# ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

# Shell used to process this Makefile
SHELL := /bin/ksh

# Shell used in wrapper (also shell that executes script).
# Must be a version of ksh93 (e.g. /usr/dt/bin/dtksh on Solaris)
WRAPPERSHELL := /bin/bash

# Compression & Uncompression Engines
COMPRESS := /bin/gzip -c # -c same as --to-stdout
UNCOMPRESS := /bin/zcat

# Encryption Engine
#CRYPT := (Replace with your stronger crypto)
CRYPT := /usr/bin/openssl enc -aes256 -a -k
DECRYPT := /usr/bin/openssl enc -aes256 -a -d -k

# Command prefixed to script to disable tracing
TRACE_DISABLE := set +x

# ### sed cheatsheet ###
# sed -e '/^$/d'# Delete blank lines from file
# sed -e 's/^[ \t]*//'# Delete leading whitespace
# sed -e 's/[ \t]*$//'# Delete trailing whitespace
## (Earlier than GNU sed 3.03 replace \t with an actual tab.)
# sed -e '/^\s*#\([^!]\|$\)/d'# Delete lines starting with "#", but not "#!"
# sed -e "1i\\" -e 'STUFF'# Insert STUFF at start of file
# sed -e "\$$a\\" -e 'STUFF'# Append STUFF at end of file

# make automatic variables:
# $@ The file name of the target of the rule.
# $< The names of the first prerequisites.
$(TARGET) : $(SOURCE)
	sed -e '/^$$/d' \
	-e 's/^[ \t]*//' \
	-e 's/[ \t]*$$//' \
	-e '/^\s*#\([^!]\|$$\)/d' $< | \
	sed -e "1i\\" -e '$(TRACE_DISABLE)' | \
	$(COMPRESS) | \
	$(CRYPT) $(TARGET) | \
	sed -e "1i\\" -e "#! $(WRAPPERSHELL)" \
	-e "1i\\" -e '$(COPYRIGHT)' \
	-e "1i\\" -e 'export script_name=\$$(/usr/bin/basename \$$0)' \
	-e "1i\\" -e '$(WRAPPERSHELL) -c "\$$(/bin/cat <<"END_OF_FILE" | $(DECRYPT) \$${0##*/} | $(UNCOMPRESS)' \
	-e "\$$a\\" -e 'END_OF_FILE' \
	-e "\$$a\\" -e ')" -- "\$$@"' >| $@
	/bin/chmod a+x $@

# *** Phony Rules ****

# ### Rule to make a debug version of a script ###
# Useful if an error occurs at a certain line number. Since the line numbers
# of the file executed don't correspond to the source file because of
# comment and blank line stripping, it is useful to have a debug version
# available for reference.
.PHONY: debug
debug :
	sed -e '/^$$/d' \
	-e 's/^[ \t]*//' \
	-e 's/[ \t]*$$//' \
	-e '/^\s*#\([^!]\|$$\)/d' $(SOURCE) | \
	sed -e "1i\\" -e '$(TRACE_DISABLE)' > $(TARGET).debug
	/bin/chmod a+x $(TARGET).debug

# ### Rule to clean up the directory ###
# The "-" at the beginning of the command tells GNU make ignore errors like
# file not present etc.
.PHONY: clean
clean :
	-/bin/rm $(TARGET)
	-/bin/rm $(TARGET).debug
