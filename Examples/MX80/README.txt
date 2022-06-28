These are example files for an Epson MX80 dot matrix printer connected
to a generic USB-parallel adapter.

It shows under "usb-devices" as

  T:  Bus=01 Lev=02 Prnt=02 Port=03 Cnt=01 Dev#=  3 Spd=12  MxCh= 0
  D:  Ver= 1.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS= 8 #Cfgs=  1
  P:  Vendor=1a86 ProdID=7584 Rev=02.54
  S:  Product=USB2.0-Print 
  C:  #Ifs= 1 Cfg#= 1 Atr=80 MxPwr=96mA
  I:  If#=0x0 Alt= 0 #EPs= 2 Cls=07(print) Sub=01 Prot=02 Driver=usblp

Which matches lsusb (Bus/Dev) of:

  Bus 001 Device 003: ID 1a86:7584 QinHeng Electronics CH340S

Which confirms the magic numbers we need are 1a86:7584.  This is a CH340S
adapter.

So to install this printer:

  cp printer.rules /etc/udev/rules.d/
  cp mx80 /usr/lib/cups/filter/mx80
  lpadmin -p mx80 -P ./mx80.ppd -v file:///dev/null -E
