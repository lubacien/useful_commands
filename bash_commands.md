Here are a few bash tricks I needed to write down at some point as I don't use
them enough to remember them. After some practice, most are obvious.

# Basic stuff
For loop syntax
```bash
# On numbers, characters, strings, etc.
for i in a b c d 5 3 g French English; do echo $i; done

# Can even be done on a digit interval:
for i in {1..10}; do echo $i; done
```

While loop on lines
```bash
cat $yourfile | while read line; do "This is your current line: $line"; done
```

# The sed tricks
```bash
# Replace windows newlines by unix newlines
sed 's/^M$//' $yourfile

# Replace newline by a space
sed ':a;N;$!ba;s/\n/ /g' $yourfile

# Merge multiple spaces into one space
sed 's/ \+ / /g' $yourfile

# Extract the string between "STRING1" and "STRING2"
sed '/STRING1/!d;s//&\n/;s/.*\n//;:a;/STRING2/bb;$!{n;ba};:b;s//\n&/;P;D' $yourfile

# Use variables inside sed command
sed -i "s/$var1/ZZ/g" $yourfile

# Add string at the end of a line containing pattern at the beginning of the line
sed '/^pattern/ s/$/ string/' $yourfile

# Replace string1 by string2 only on line 5
sed -e "5s/string1/string2/" $yourfile

# Remove empty lines (with spaces, tabs, or truly empty)
sed '/^\s*$/d' $yourfile

# Replace the first space by a tab on each line
sed 's/ /'$'\t''/' $yourfile
```

# The awk tricks
```bash
# Sum the number of one column (here the first one) in a file
awk '{ sum+=$1} END {print sum}' $yourfile

# To get all the lines between pattern1 and pattern2 (included) in a file
awk '/pattern1/ { show=1 } show; /pattern2/ { show=0 }' $yourfile

# Print only lines with less than X characters
awk 'length < X' $yourfile

# Print columns between 3 and 12 (included)
awk '{for(i=3;i<=12;++i)print $i}'
# or:
awk '{for(i=3;i<=12;i++){printf "%s ", $i}; printf "\n"}'

# "Transpose" a file (convert row to column and vice versa)
awk '
{
    for (i=1; i<=NF; i++)  {
        a[NR,i] = $i
    }
}
NF>p { p = NF }
END {
    for(j=1; j<=p; j++) {
        str=a[1,j]
        for(i=2; i<=NR; i++){
            str=str" "a[i,j];
        }
        print str
    }
}' $yourfile

# Use variables in awk
awk -v var="$variable" 'BEGIN {print var}'

# Use multiple if conditions
awk '{if ($3 =="" || $4 == "" || $5 == "") print $1, "Missing col 2"}' $yourfile
```
# The grep tricks
```bash
# Grep stuff before newline
grep "stuff$" $yourfile

# Grep capitalised words (with more than 3 letters):
grep -oE "\b[[:upper:]]+[[:upper:]\ '\-]+[[:upper:]]\b" $yourfile

# Grep something with potential dash (-) in it
grep [options] -- "$pattern" $yourfile
```

# The sox tricks (audio manipulation)
```bash
# Convert from raw to wav (48k)
sox -r 48000 -e signed -b 16 -c 1 $yourfile $youroutput

# Downsample with sox (to 16k)
sox -G $yourfile -r 16000 $youroutput
```

# Other various tricks
```bash
# Check if directory is empty or not
if [ "$(ls -A /path/to/dir)" ]; then echo "Not Empty"; else echo "Empty"; fi

# Negative if
if ! [ 0 == 2 ]; then echo Hello; fi

# Remove last  X lines of a file
head -n -X $yourfile

# Replace spaces by newlines in a file
tr ' ' '\n' < $yourfile

# Or for tabulations:
tr '\t' '\n' < $yourfile

# "cat" without the last "end of line"
head -c -1 $yourfile

# To find all empty files in subdirectories up to some depth
find -L $yourdirectory -maxdepth 1  -type f -size 0

# Print 50 directories with most files:
find /yourdirectory/ -xdev -type d -exec sh -c '
  echo "$(find "$0" | grep "^$0/[^/]*$" | wc -l) $0"' {} \; |   sort -rn |   head -50

# To merge 2 files with columns (eg col1 col2 in $file1 and col3 in $file2 > col1 col2 col3)
paste $file1 $file2 > $file3
```