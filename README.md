# vhdl-cheatsheet
summarizes most important things to recognize about vhdl

# using stuff from librarys
```vhdl
library IEEE;
use IEEE.std_logic_1164.all;
```

# creating an entity
```vhdl
entity multiplexer is
  port(
    a, b: in std_logic;
    s: in std_logic;
    c: out std_logic
  );
end entity;
```

# creating an architecture
```vhdl
architecture multiplexer_logic of multiplexer is
begin
  c <= a when s = "0" else
       b when s = "1";
end multiplexer_logic;
```

# simple assignments
```vhdl
a <= "1";
```

# when else assignments
```vhdl
a <= "11" when b = "00" else
     "10" when b = "01" else
     "01" when b = "10" else
     "00" when b = "11";
```


# with select assignments
this is equal to the code above
```vhdl
with b select a <= 
     "11" when "00",
     "10" when "01",
     "01" when "10",
     "00" when "11";
```
