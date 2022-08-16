Using SESAME EOS database in FLASH
These are just supplymentary notes. See [opacplot2][https://github.com/flash-center/opacplot2] github for more details.
Note: There was a bug in one the scripts for .tops > .cn4 which I changed. You can install this updated version from [here][https://github.com/PhysicsDan/opacplot2] by running

pip install git+https://github.com/PhysicsDan/opacplot2

Get access to the [SESAME database][https://www.lanl.gov/org/ddste/aldsc/theoretical/physics-chemistry-materials/sesame-database.php]
Search through the sesame.ascii file to find the 'code' for your element
I recommend if you're using mac/linux using [less][https://www.tutorialspoint.com/unix_commands/less.htm] because the file is pretty long. You can then search by typing \boron and hitting enter to cycle through results. 
Note down the number in the row below (it should be 4 digits like 2330 for boron). Type q and enter to quit less
To extract your EOS from the database you must first use the sesame_extract.py script. This will make a .ses file which is used in the next step
E.g. python sesame_extract.py -o boron.ses sesame.ascii 2330
Now you can use the opac_convert.py scipt to create the .cn4 file
E.g. python opac_convert.py --Znum 5 -i sesame boron.ses
You can get opacity data from the [TOPS Opacities][https://aphysics2.lanl.gov/apps/] database. Make sure to select 'Multigroup opacities' and numerical by columns as shown in the images below otherwise the opac_convert 
code won't work. When the webpage with the data appears just save it as a .html file. The opac_convert.py script can recognise this.

Then run the code
`python opac_convert.py --outname output.cn4 input.html'

![[img/TOPS_output_data_type.png]]
![[img/TOPS_output_table.png]]
