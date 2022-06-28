# Dumb USB printing

If you're like me then you may have difficulty in getting CUPS to print
to USB connected printers.  Especially the "dumb" variety (eg an old
school dot matrix printer connected via a USB/Parallel adapter).

In theory CUPS should autodetect the printer, so the WebUI will let you
install it, but sometimes this... fails.

You can see it with the command `/usr/lib/cups/backend/usb` but it returns
output such as

    direct usb://Unknown/Printer?serial=Printer "Unknown" "Unknown" "	 " ""

Not helpful!

So I wrote this kludge that lets us abuse the interface script to print
directly to the `/dev/usb/lp*` entry.

### Warning

I've come a cross one adapter, Prolific based, that won't let you do this;
multiple attempts to write to `/dev/usb/lp*` are... dropped.  The first
attempt appears to work but then it just stops working.

I've had success with CH340S and Winbond devices

## Find the printer

Because the `/dev/usb/lp*` are dynamic you may not end up with the same
address each time; especially if you have multiple USB printers.  So what
we can do is use `udev` to assign a known fixed name.

For me, I'm using the USB VID:PID values, which can be found in a number
of ways

### Option 1.

Use the `usb-devices` command and look for an entry similar to
 
    T:  Bus=05 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  9 Spd=12  MxCh= 0
    D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
    P:  Vendor=0416 ProdID=5011 Rev=02.00
    S:  Manufacturer=STMicroelectronics
    S:  Product=POS58 Printer USB       
    S:  SerialNumber=Printer
    C:  #Ifs= 1 Cfg#= 1 Atr=c0 MxPwr=100mA
    I:  If#=0x0 Alt= 0 #EPs= 2 Cls=07(print) Sub=01 Prot=02 Driver=usblp

The import part is that it uses `Driver=usblp`.  The Manufacturer and Product
entries can also help you verify you have the right device.

The important numbers we want are the Vendor and prodID values.

### Option 2.

Use `lsusb -t` command and look for the usblp entry.
e.g
 
    /:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=ohci-pci/3p, 12M
     |__ Port 1: Dev 9, If 0, Class=Printer, Driver=usblp, 12M

So this tells us we need Bus 5 (the first line) and Device 9 (the
second line).  Now a normal `lsusb` will have this as part of the output

    Bus 005 Device 009: ID 0416:5011 Winbond Electronics Corp. Virtual Com Port

We want the two numbers after the ID (that's the vendor and prodid)

### Option 3.

Turn off the printer.  Turn it back on again.  This should cause
a new entry in the kernel log, such as

    % sudo dmesg | grep usblp | tail -1
    [12595.482773] usblp 5-1:1.0: usblp1: USB Bidirectional printer dev 9 if 0 alt 0 proto 2 vid 0x0416 pid 0x5011

The VID and PID values are what we want.

### Update the printer.rules and udev

However you find your printer VID:PID values you need to insert them into
the `printer.rules` script.  At the same time pick a good name for your
printer device.

eg

    #Rules for Terow printer
    KERNEL=="lp*" ATTRS{idVendor}=="0416", ATTRS{idProduct}=="5011", MODE="0666", SYMLINK+="lp_receipt"

You can now copy this to the `udev` tree:

    cp printer.rules /etc/udev/rules.d/

This should take immediate effect, so turn off the printer and turn it
on again.

You should now see your printer in the `/dev` directory; e.g.

    % ls -l /dev/lp_receipt 
    lrwxrwxrwx 1 root root 7 Jun 28 18:25 /dev/lp_receipt -> usb/lp0

## Define the printer in CUPS

Edit the `dumb` script to refer to the correct `/dev` entry you just
create.  Now you can copy the file to the `filter directory

    cp dumb /usr/lib/cups/filter/

And now you can create the printer

    lpadmin -p printer_name -P ./dumb.ppd -v file:///dev/null -E

### Slightly advanced, optional

You can call the file `dumb` a different name (e.g. in my case I called
it `receipt`), and also updated `dumb.pdd` to use the new name (e.g. rename
it to `receipt.ppd` and update the `dumb` entries inside the `ppd` to the new
name).  Now you can do something like

    cp receipt /usr/lib/cups/filter/
    lpadmin -p receipt -P ./receipt.ppd -v file:///dev/null -E

## Check the printer is defined

Now "lpstat -t" should show a printer called "printer_name" (e.g. `receipt`)

    % lpstat -t
    scheduler is running
    system default destination: receipt
    device for receipt: ///dev/null
    receipt accepting requests since Tue Jun 28 18:10:49 2022
    printer receipt is idle.  enabled since Tue Jun 28 18:10:49 2022

You may need to run the `accept` and `cupsenable` commands if the status
shows as "Rejecting jobs" or "paused".

you should be able to print to it!

e.g.

    % echo hello | lp -dreceipt

## Notes

This is a DUMB printer.  It won't try and convert graphics, or postscript
or anything.  It will send to the printer EXACTLY what you tell it to.
If you try to send anything clever (eg a postscript file) then it might
fail with

     lp: Unsupported document-format "application/postscript".

But this is sufficient for dumb text printing!

## Examples

The Examples directory contains configurations for my Terow label printer
(the `receipt` script tries to reset the printer before the job start and
sends an additional blank line afterwards), and for hooking up an old
Epson dot matrix printer.

These may help explain some of the customisations that can be done.

You can [see it working](https://www.youtube.com/watch?v=LioWDLqEs04) in a very retro setting, printing from a BBC Micro via Econet to an MX80 using a Raspberry Pi
