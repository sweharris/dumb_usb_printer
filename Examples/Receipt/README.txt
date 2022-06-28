These are example files for a Terow thermal Receipt printer

It shows under "usb-devices" as

  T:  Bus=05 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  9 Spd=12  MxCh= 0
  D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
  P:  Vendor=0416 ProdID=5011 Rev=02.00
  S:  Manufacturer=STMicroelectronics
  S:  Product=POS58 Printer USB       
  S:  SerialNumber=Printer
  C:  #Ifs= 1 Cfg#= 1 Atr=c0 MxPwr=100mA
  I:  If#=0x0 Alt= 0 #EPs= 2 Cls=07(print) Sub=01 Prot=02 Driver=usblp

Which matches lsusb (Bus/Dev) of:

  Bus 005 Device 009: ID 0416:5011 Winbond Electronics Corp. Virtual Com Port

Which confirms the magic numbers we need are 0416:5011.  This is a Winbond
adapter.

So to install this printer:

  cp printer.rules /etc/udev/rules.d/
  cp receipt /usr/lib/cups/filter/receipt
  lpadmin -p receipt -P ./receipt.ppd -v file:///dev/null -E
