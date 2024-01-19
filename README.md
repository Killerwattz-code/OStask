# Dynatrace to Elastic Common Scheme (ECS) mapping rules

#### Rules

Steps:
1. Get string value in each row of CSV file by identifying the 'alias\~os\~version' header and extracting the string value associated with it.
2. Remove carriage return and newline characters from the extracted string value.
3. Remove white spaces from start and end of the extracted string value.
4. Detect 'os\~family' from 'alias\~os~version' string value by using the regex pattern:

(?i)(red hat|debian|freebsd|windows|centos|ubuntu|amazon|sunos|z/os)

Which searches for any of the words in the given string ignoring case sensitivity. If found the first match will give us the 'os\~family' value. The value will need to be cleaned of any extra whitespace and characters like '/' , followed by changing to lower case as per requirements. Transforming 'z/OS' for example into 'zos'.
Having this value first is important as it sets the rules for finding other values in the string.

5. 'os\~name' is mapped to the same value as 'os\~family'.

6. To process 'os~\kernel' we use the regex pattern:

kernel\s*\S+

this grabs the word 'kernel' followed by any number of spaces and then any number of non white space characters. From that we check if the value has ')' at the end and if so remove it. If no match is found we return 'None' as there isn't enough information to determine the kernel.

7. To process 'os\~architecture' we use the regex pattern:

(?i)(x86_64|ppc64le)

on the 'alias\~os~version' value , this pattern will detect if there is an acceptable architecture. We then return the accepted value if detected otherwise return nothing. 

8. For 'user\~product' we just set it to '['dynatrace']' because this extract is from Dynatrace.

9. For 'os~version' we use the regex pattern: 

\d*\d\.\d\d*\.*\d*

to extract the entire version number from 'alias\~os\~version'. 

10. For the 'user\~os\~version\~major', 'user\~os\~version\~minor' and'user\~os\~version\~patch' we take the extract from 'os\~version' and split it at each '.' , the first item is the 'major' value, the second item is the 'minor' value and if theres a third item that is the 'patch' value.

11. To process 'os\~full' we use the regex pattern: 

^(.*?)(?:\s*\(*kernel|$)

on the 'alias\~os~version' value. The first match from this extract is the full string value with kernel information excluded.

12. To process 'os\~type' we can use the 'alias\~os\~Type' value and set it to lower case as per requirements.

13. To process 'os\~platform' we can use the same value as 'os\~type', however in the case of 'os\~type' being equal to 'linux', we instead use the 'os\~family' value to be more accurate. 
