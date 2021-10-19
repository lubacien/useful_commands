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

# Add a space between each character (also see awk version below)
sed 's/./& /g' $yourfile
```

# The awk tricks
this will change a transcriptions file per segments to per conv ids:
```bash
awk '{split($1,a,"_"); q=a[1]; $1=""; sentence[q] = sentence[q] $0 }END{for (i in sentence) { print i " "sentence[i]}}' yes_transcript > yes_transcript_convid
```

this will change a text file into a spk2utt file in kaldi: (but remove the first line after)
```bash
gawk 'BEGIN{split($1,a,"_"); spk = a[1]; prev= a[1]} {split($1,b,"_"); if (b[1] != spk) {printf prev " "; for (i in arr) {printf i " "}; print " "; delete arr; spk= b[1]; } arr[$1];prev=b[1] }' text > spk2utt
```

```bash
(almost) change language model to upper case, (but actually still need to change 2-grams and 3-grams)
cat LM.arpa | awk 'NR<=11{print $0; next}NR>11{print toupper($0)}' > LM_UC.arpa

# Sum the number of one column (here the first one) in a file
awk '{ sum+=$1} END {print sum}' $yourfile

# To get all the lines between pattern1 and pattern2 (included) in a file
awk '/pattern1/ { show=1 } show; /pattern2/ { show=0 }' $yourfile

# Print only lines with less than X characters
awk 'length < X' $yourfile

# Print columns between 3 and 12 (included)
awk '{for(i=3;i<=12;++i)print $i}' $yourfile
# or:
awk '{for(i=3;i<=12;i++){printf "%s ", $i}; printf "\n"}' $yourfile

# Add a space between each character
awk '$1=$1' FS= OFS=" " $yourfile

# Match 2 files based on a column
# This command will print the full lines of $file_where_to_look_in
# if the first field of $what_you_are_looking_for and $file_where_to_look_in match
#NR is 
awk 'NR==FNR{a[$1]; next}$1 in a{print $0}' $what_you_are_looking_for $file_where_to_look_in

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

# Use printf (don't forget "\n")
awk '{printf("This is column 1: %s, this is column 3: %s, and this is column 5: %s\n", $1, $3, $5)}'
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

# Audio manipulation
## sox
```bash
# Convert from raw to wav (48k)
sox -r 48000 -e signed -b 16 -c 1 $yourfile $youroutput

# Downsample with sox (to 16k)
sox -G $yourfile -r 16000 $youroutput

# Get info on audio file
soxi $audiofile

# Trim an audio file (from 1 sec to 6.2 sec)
sox $audiofile $outputaudiofile trim 1.0 5.2

# Convert raw (mono ulaw 8kHz) to wav (mono pcm 16bit 8kHz)
sox -t ul -r 8000 -c 1 in.raw -r 8000 -b 16 -c 1 out.wav
```

## ffmpeg
```bash
# need to install ffmpeg
# Simple conversion from mp3 to wav
ffmpeg -i input.mp3 output.wav

# Conversion to specific wanted format (here 16 bits 16kHz mono)
ffmpeg -v 8 -i input.mp3 -f wav -acodec pcm_s16le -ac 1 -ar 16000 output.wav
```

# Archive extraction
```bash
# .tar.bz2
tar -xvf archive.tar.bz2

# .tar.gz
tar -xvzf archive.tar.gz

# .gz
gunzip -c text.txt.gz > text.txt
gunzip < text.txt.gz > text.txt

# .zip
unzip archive.zip

# Compression to .tar.gz
tar cvzf archive.tar.gz archive

# Extracting multi sub-archives (zip)
# where you have archive.zip.001 archive.zip.002 archive.zip.003 ...
cat archive.zip.* > archive.zip && unzip archive.zip
```

# Docker related
```bash
# Remove dangling images
docker image prune

# Remove all stopped containers
docker container prune

# Prune volumes
docker volume prune

# Prune everything (images, containers, networks)
docker system prune
# and to also prune volumes at the same time:
docker system prune --volumes

# Source: https://docs.docker.com/config/pruning/
```

# Other various tricks
```bash

#compare 2 files. column 1 is files unique to 1, 2 is for 2nd and 3rd is the files that appear in both. - is to hidethe specific column.
comm -23 <(sort a.txt) <(sort b.txt)

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

# Sort lines by length
cat $yourfile | awk '{ print length, $0 }' | sort -n -s | cut -d" " -f2-

# Write to file from bash script
cat << EOF > $your_file
This will be the content
of the file you wanted to
create, even with variables
like $var1
EOF

# A version where variables are not interpreted:
cat << 'EOF' > $your_file
This will still show
$var1 and not the content of
$var1. It can be useful to "write"
scripts.
EOF
```
