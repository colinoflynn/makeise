MakeISE: Saving your Soul when using Xilinx Projects in GIT
===========================================================

This is a simple script to automatically generate ISE project files from a commong input format. Features include:

* Simple text-based entry format avoids git conflicts on .XISE projects
* Generates COREGen files
* Ability to change package/device within same family and automatically updated project file + coregen file
* Progmatically generated Verilog file can be used as a 'setup' file which enables/disables features, allows use of a common code-base across multiple hardware versions

If you've ever been involved in a project that requires you to support different hardware, you'll understand what a hassle the Xilinx project files are. They require you to manually recreate the COREGen files when changing even device package! This project hopes to solve these issues.

You can see full details in the February 2015 issue of Circuit Cellar, which describes this project in detail.

Background on ISE File Formats
==============================

As much as you want to ignore this, some knowledge of the project files you are generating is required. This section briefly outlines the project files (.xise) and coregen files (.xco).

Project Files
-------------

The required project file is the one with the .xise extension, from that a .gise file is generated. The .xise file is a simple XML-based format, the following is an example of these files:

```
  <version xil_pn:ise_version="14.6" xil_pn:schema_version="2"/>

  <files>
   <file xil_pn:name="interface.v" xil_pn:type="FILE_VERILOG">
      <association xil_pn:name="BehavioralSimulation" xil_pn:seqID="1"/>
      <association xil_pn:name="Implementation" xil_pn:seqID="3"/>
    </file>
  </files>

  <properties>
    <property xil_pn:name="AES Initial Vector spartan6" xil_pn:value="" xil_pn:valueState="default"/>
```

Coregen Files
-------------

The COREGen file required is simply the .xco file - from this a number of other files are generated. When you attempt to synthesize a project with just the .xco file, it will ask you to generated the remaining files automatically, which take some time. Encoded in the .xco file is details of the device along with details of your block - meaning you can't copy .xco files between projects with different devices/packages!

```
...
SET createndf = false
SET designentry = Verilog
SET device = xc6slx25
SET devicefamily = spartan6
SET flowvendor = Other
SET formalverification = false
SET foundationsym = false
SET implementationfiletype = Ngc
SET package = ftg256
...
CSET clock_enable_type=Slave_Interface_Clock_Enable
CSET clock_type_axi=Common_Clock
CSET component_name=fifoonly_adcfifo
CSET data_count=false
CSET data_count_width=13
CSET disable_timing_violations=false
CSET disable_timing_violations_axi=false
CSET dout_reset_value=0
CSET empty_threshold_assert_value=4
CSET empty_threshold_assert_value_axis=1022
...
```

Example MakeISE Project File and Run
====================================

The following shows an example project file, each section will be discussed in detail:

```
[ISE Configuration]
InputFile = ise_verilog_template.xise.in
Version = 14.4
Device Family = Spartan6
Package = ftg256
Device = xc6slx25
Speed Grade = -3
Verilog Include Directories = ../../../hdl|../../refproject
Other Map Command Line Options = -convert_bram8

[UCF Files]
system.ucf

[Verilog Files]
#Can have comments too anywhere
simpletop.v
simplemodule.v
setup.v = Setup File

[CoreGen Files]
fifoonly_adcfifo.xco = ADC FIFO CoreGen Setup

[ADC FIFO CoreGen Setup]
InputFile = fifoonly_adcfifo.xco.in
input_depth = 8192
output_depth = CALCULATE $input_depth$ / 4
full_threshold_assert_value = CALCULATE $input_depth$ - 2
full_threshold_negate_value = CALCULATE $input_depth$ - 1
write_data_count_width = 16
read_data_count_width = 16
data_count_width = 16

[Setup File]
BOARD_REV2
UART_CLK = 40000000
UART_BAUD = 512000
```
__[ISE Configuration]__

This section uses an existing ISE Project file as a 'template'. Typically you can just create an empty ISE project as a template, and from there various settings will be twiddled.

If you want to change a setting simply find the name in the .xise file, and add it here. This allows you to override almost anything, in this example I've added some include directories and modified special command-line MAP options.

The device type, package, and speed grade are treated specially. They will also be used when generating the COREGen files.

__[UCF Files]__

This section will result in a .ucf file being added to your project. Note the system will simply generate a project file with the given UCF path - it's up to you to ensure the file is present!

__[Verilog Files]__

Like the .ucf files, these are added to the project blindly. The only special note is that anything with a "=" sign means an auto-generated file. In this example setup.v will automatically be generated from the [Setup File] section.

__[CoreGen Files]__

You can simply add existing .xco files which are properly setup, ala Verilog files or UCF files. But it also supports automatically generated COREGen files, which is one of the most useful aspects of this project.

Automatically generated files are specified by the format `filename.xco=[Section Name]`. This file will be generated and copied to the appropriate location. Details of the auto-generation will be discussed next.

__[ADC FIFO CoreGen Setup]__

This is an example of an automatically generated COREGen file. We use `fifoonly_adcfifo.xco.in` as an input - this is a COREGen file which I've setup via the GUI. The part number, package, and device speed will be **automatically** changed in the resulting .xco file based on settings given in the `[ISE Configuration]` section.

You can additionally modify settings as required - note you can perform calculations based on settings as shown in the example. You need to pay special attention to what sort of attributes are linked together, as it's easy to generate invalid files!

Check this by using the GUI to tweak your desired parameters, and seeing what changes. You can also refer to the datasheet for the COREGen module for details of the parameters.

__[Setup File]__

This section is used for an automatically generated setup.v file. 


Running an Example
==================

Using MakeISE in your own Project
=================================

