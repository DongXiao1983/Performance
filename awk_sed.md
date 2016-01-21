5.5.4 Examples Using printf

The following simple example shows how to use printf to make an aligned table:

awk '{ printf "%-10s %s\n", $1, $2 }' mail-list

This command prints the names of the people ($1) in the file mail-list as a string of 10 characters that are left-justified. It also prints the phone numbers ($2) next on the line. This produces an aligned two-column table of names and phone numbers, as shown here:

$ awk '{ printf "%-10s %s\n", $1, $2 }' mail-list
-| Amelia     555-5553
-| Anthony    555-3412
-| Becky      555-7685
-| Bill       555-1675
-| Broderick  555-0542
-| Camilla    555-2912
-| Fabius     555-1234
-| Julie      555-6699
-| Martin     555-6480
-| Samuel     555-3430
-| Jean-Paul  555-2127

In this case, the phone numbers had to be printed as strings because the numbers are separated by dashes. Printing the phone numbers as numbers would have produced just the first three digits: ‘555’. This would have been pretty confusing.

It wasn’t necessary to specify a width for the phone numbers because they are last on their lines. They don’t need to have spaces after them.

The table could be made to look even nicer by adding headings to the tops of the columns. This is done using a BEGIN rule (see BEGIN/END) so that the headers are only printed once, at the beginning of the awk program:

awk 'BEGIN { print "Name      Number"
             print "----      ------" }
           { printf "%-10s %s\n", $1, $2 }' mail-list

The preceding example mixes print and printf statements in the same program. Using just printf statements can produce the same results:

awk 'BEGIN { printf "%-10s %s\n", "Name", "Number"
             printf "%-10s %s\n", "----", "------" }
           { printf "%-10s %s\n", $1, $2 }' mail-list

Printing each column heading with the same format specification used for the column elements ensures that the headings are aligned just like the columns.

The fact that the same format specification is used three times can be emphasized by storing it in a variable, like this:

awk 'BEGIN { format = "%-10s %s\n"
             printf format, "Name", "Number"
             printf format, "----", "------" }
           { printf format, $1, $2 }' mail-list



    sub(/^[[:blank:]]*/,"",变量)  是去掉变量左边的空白符
    sub(/[[:blank:]]*$/,"",变量) 是去掉变量右边的空白符
    gsub(/[[:blank:]]*/,"",变量) 是去掉变量中所有的空白符

    示例：
    echo ' 123 456 789  ' | awk '{
    print "<" $0 ">";
    sub(/^[[:blank:]]*/,"",$0);print "[" $0 "]";
    sub(/[[:blank:]]*$/,"",$0);print "|" $0 "|";
    gsub(/[[:blank:]]*/,"",$0);print "/" $0 "/";
    }'
