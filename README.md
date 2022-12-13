# BigSpicy

This repo shows the steps for merging the SPEF, verilog and spice netlist into a circuit protobuf and generating the spice file of the design which can further be used to perform various tests and analysis.
In this repo, I have used the design of Uart Transmitter implemented using SKY130 PDKS. The RTL to GDS2 flow of the given design can be referred from the following github repo.
 https://github.com/jayshah1x/iiitb_uarttx.git
 
 # Flowchart
 
 
 ![flowchart](https://user-images.githubusercontent.com/46132046/207431170-df86674a-2be0-4ea3-80f2-ac0ed2da4154.png)

# Prerequisites

To install the python dependencies, follow the below steps:

```
git clone https://github.com/jayshah1x/iiitb_uarttx.git
cd BigSpicy/
sudo apt-get update
pip install -e ".[dev]"
pip install -r requirements.txt
sudo apt install -y protobuf-compiler iverilog
```

Another prerequisite for this step is to compile protobufs into python file.(_pb2.py).
To compile the protobufs, type the below command in terminal in the BigSpicy(cloned_repo) directory:

```
git submodule update --init  
protoc --proto_path vlsir vlsir/*.proto vlsir/*/*.proto --python_out=.
protoc proto/*.proto --python_out=.
```

We also need tools like Xyce and XDM installed.
To install the mentioned tools use the following links:
XYCE:
https://syce.scandia.gov/documentation/BuildingGuide.html

XDM:
https://github.com/XYCE/XDM


# Converting the PDKs:

First step is to convert the PDKs into Xyce format.
To convert the PDK's go to the directory where XDM is installed and type the following command:

```
xdm_bdl -s hspice "path to the pdk"/"file to be converted" -d lib
```

# Merging 

We merge the files into circuit protobuf(final.pb) which is used to generate the whole module spice models and to conduct the various tests using Xyce.
To merge the files, follow the below steps in the BigSpicy directory:

```
./bigspicy.py \
   --import \
   --spef example_inputs/iiitb_uarttx/iiitb_uarttx_rc.spef \
   --spice lib/sky130_fd_sc_hd.spice \
   --verilog example_inputs/iiitb_uarttx_rc/iiitb_uarttx_rc.v \
   --spice_header lib/sky130_fd_pr__pfet_01v8.pm3.spice \
   --spice_header lib/sky130_fd_pr__nfet_01v8.pm3.spice \
   --spice_header lib/sky130_ef_sc_hd__decap_12.spice \
   --spice_header lib/sky130_fd_pr__pfet_01v8_hvt.pm3.spice \
   --top iiitb_uarttx_rc \
   --save final.pb \
```

This will generate final.pb file.
To specify the location of the final.pb file, go to bigspicy.py file and search for "def withoptions()" function. Change the "output_dir" variable to your desired path.

# Generating Whole-module spice file

After generating the "final.pb" file, we now generate the spice file("spice.sp" in this case) for our design which can be further used to run tests.
This step takes the pdks, and the design as input and gives the spice file as output.
To generate the spice file, follow the below steps in BigSpicy directory:

```./bigspicy.py --import \
    --verilog example_inputs/iiitb_uarttx/iiitb_uarttx.v \
    --spice lib/sky130_fd_sc_hd.spice \
    --spice_header lib/sky130_fd_pr__pfet_01v8.pm3.spice \
    --spice_header lib/sky130_fd_pr__nfet_01v8.pm3.spice \
    --spice_header lib/sky130_ef_sc_hd__decap_12.spice \
    --spice_header lib/sky130_fd_pr__pfet_01v8_hvt.pm3.spice \
    --save final.pb \
    --top iiitb_uarttx \
    --flatten_spice --dump_spice spice.sp


```

The above steps will generate "spice.sp" file in the mentioned directory.

# Future Works

We are currently working on testing the generated spice file "spice.sp".
Then we have to determine the path delay for our design through Xyce test and for this test we need to list down the different paths using xyce and compare the delay results with other available tools.


# Acknowledgements


* Kunal Ghosh, Director VSD Corp. Pvt. Ltd.
* Dr. Nanditha Rao, IIIT Bangalore
* Dr. Madhav Rao, IIIT Bangalore
* Arya Reais-Parsi, PhD Candidate at University of California, Berkeley

