# Counters

CmRDT counters are rather trivial.
Each operation contains a number, either positive or negative.
The resulting state is a sum of those numbers.

The only op type is `.add`.
Op or state value is a decimal number, possibly preceded with '+' or '-' for positive or negative values, e.g. `+1`.
