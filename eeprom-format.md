# B+ HAT ID EEPROM FORMAT SPECIFICATION

**CURRENTLY THIS SPEC IS PRELIMINARY AND THEREFORE STILL LIKELEY TO CHANGE**

Note that there are [software tools](./eepromutils) for creation of EEPROM images and for flashing / dumping images from attached HATs.

## EEPROM Structure

```
  HEADER  <- EEPROM header (Required)
  ATOM1   <- Vendor info atom (Required)
  ATOM2   <- GPIO map atom (Required)
  ATOM3   <- DT blob atom (Recommended)
  ...
  ATOMn
```

## EEPROM Header Structure

```
  Bytes   Field
  4       signature   signature: 0x52, 0x2D, 0x50, 0x69 ("R-Pi" in ASCII)
  1       version     EEPROM data format version (0x00 reserved, 0x01 = first version)
  1       reserved    set to 0
  2       numatoms    total atoms in EEPROM
  4       eeplen      total length in bytes of all eeprom data (including this header)
XXX CRC16???

```

## Atom Structure
```
  Bytes   Field
  2       type        atom type
  2       count       incrementing atom count
  4       dlen        length in bytes of data+CRC
  N       data        N bytes, N = dlen-2
  2       crc16       CRC-16-CCITT of entire atom (type, count, dlen, data)
```

## Atom Types

```
  0x0000 = invalid
  0x0001 = vendor info
  0x0002 = GPIO map
  0x0003 = Linux device tree blob
  0x0004 = manufacturer custom data
  0x0005-0xfffe = reserved for future use
  0xffff = invalid
```

### Vendor info atom data (type=0x0001):

```
  Bytes   Field
  4       serial      product serial number
  2       pid         product ID
  2       pver        product version
  1       vslen       vendor string length (bytes)
  1       pslen       product string length (bytes)
  X       vstr        ASCII vendor string e.g. "ACME Technology Company"
  Y       pstr        ASCII product string e.g. "Special Sensor Board"
```

### GPIO map atom data (type=0x0002):

  GPIO map for bank 0 GPIO on 40W B+ header.

```
  Bytes   Field
  1       bank_drive  bank drive strength/slew/hysteresis, BCM2835 can only set per bank, not per IO
            Bits in byte:
            [3:0] drive       0=leave at default, 1-8=drive*2mA, 9-15=reserved
            [5:4] slew        0=leave at default, 1=slew rate limiting, 2=no slew limiting, 3=reserved
            [7:6] hysteresis  0=leave at default, 1=hysteresis disabled, 2=hysteresis enabled, 3=reserved
  1       reserved    set to 0
  28      1 byte per IO pin
            Bits in each byte:
            [2:0] func_sel    GPIO function as per FSEL GPIO register field in BCM2835 datasheet
            [4:3] reserved    set to 0
            [6:5] pulltype    0=leave at default setting,  1=pullup, 2=pulldown, 3=no pull
            [  7] is_used     1=board uses this pin, 0=not connected and therefore not used
```
XXX define / link to mapping between GPIO pin numbers and the
pins on the connector. 

### Device tree atom data (type=0x0003):

Binary DT blob fragment for board hardware.

XXX todo.
