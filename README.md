MakeISE: Saving your Soul when using Xilinx Projects in GIT
===========================================================

This is a simple script to automatically generate ISE project files from a commong input format. Features include:

* Simple text-based entry format avoids git conflicts on .XISE projects
* Generates COREGen files
* Ability to change package/device within same family and automatically updated project file + coregen file
* Progmatically generated Verilog file can be used as a 'setup' file which enables/disables features, allows use of a common code-base across multiple hardware versions


You can see full details in the February 2015 issue of Circuit Cellar, which describes this project in detail.

Background on ISE File Formats
==============================

Project Files
-------------

Coregen Files
-------------

Example MakeISE Project File and Run
====================================


Using MakeISE in your own Project
=================================

