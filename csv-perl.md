# CSV

## Read CSV

```perl
use Text::CSV_XS qw[];

my $data = Text::CSV_XS::csv(
    in         => $tsv,
    sep_char   => "\t",
    quote_char => q["],      # undef
    headers    => 'auto',    # undef to return ArrayRef[ArrayRef]
);
```

## Write CSV

```perl
use Text::CSV_XS qw[];

Text::CSV_XS::csv(
    in          => Array[HashRef],
    out         => 'filename.csv', # \my $csv
    headers     => [qw[field1 field2 ...]],
    binary      => 1,
    encoding    => 'UTF-8',
    decode_utf8 => 0,
    eol         => "\n",
);
```
