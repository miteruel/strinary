# strinary
Optimize the memory uses  in Delphi strings

¿ can a file be read in memory, using less memory space  than in DISK?

Sorry for my bad english. If you understand the question, thank you.

I think  your first response maybe was NO. At least it uses the same space.
Next,  you can think than if you compress the de data in memory it uses less space.
But then why don’t have compresed the file directly, it will be more effective. 
Anyway if you need to acces to this memory data then you must decompress the info, even if uses a buffer, you spent  more memory…

Yes, you can read partial buffers of zipfile and do something with the data, but the question was read all the file in memory.

At the moment I don’t want to talk about file types, compression, mpeg, etc…
I want focus in text files, XML, JSON….

We uses a lot of XML tables with technical information to calculate the risk and cost  of some special Insurances (farming related)., without convert to db tables or to other possible more efficient format. Much of this files, have 10, 20 Mg, but there is other huge files with 200, 400 MG. Even 1GB of xml files…

Really we split this huge files, in several little files, but at least 1Mb files is very normal to work with. The program shares this “objects” in memory between different user session.
The ajax responses will be rendered, by filtering info in this XML, and locking for the correct match. The ajax request and response are XML (and JSON) too. It will be in continuous  parse-read and render-write XML operation in documents.

Because it works with all type of agricultural products, with lot of technical specific info,  This info changes at least 2 or 3 times in the week, lot of files changes the content or even the schema. Is hard to include in the main database ; it will be in continuous update.
Because some tables have thousand or even million of records, if we put it in the main database, we must request to it a lot of times, with complex sql conditions,  so why put it in the db, for then ask it with SQL, and then with the response write in xml.?

Was better work all possible directly with the original XML, and not create copys or include in user db, (with the problem of actualization,.., etc). So we need optimal components which works in efficient way with XML and/or Json.

Now, I don’t know if was the right path. The info grown, the most of the time the server do the parsing of XML documents, filtering info, and rendering new XML. But the other solution was to use a traditional SQL tables, but in this case, the server was every time doing sql request and/or buffering request to faster access,

Rational memory uses.

There is a lot of components to work with XML.  But we have created ourself components to work with  XML files, adapted to the this specific problem. It have pros and cons like the other libraries...    

    Our main objective was the speed. With a correct memory consumption.
Now the objective is to  maintain the speed, but do a more rational use of the memory.

    This “rational use” have at least 2 different aspects:
    One is minimize the memory consumption of the data readed. 
    Other is to minimize the memory allocation and free/release operations. In a web server the memory management can be a bottleneck,  when several concurrent process do continuous  allocate and release memory operation.So if we reduce this kind of operation, the process work better in several threads. And we have better general performance .

    Quick string pascal history.

    Use pascal string, was some special,  in all turbo pascal and delphi versions,     
    In the begin, the pascal had not string. The way to describe a string was:

    var s:array [1..80] of char;

You could write it 0 based:
    
    var s:array [0..79] of char;
    
But was normal, 0 based but used in position 1 and next, the 0 for collect  possible errors or info like the length of the string..

     var s:array [0..80] of char;

    The first string implementation had this structure. I don’t sure, but I remember than the first tp version I knew (Tp 2.0) the string had 80 chars by default. At least in Tp 3.0  was very usual to declare a String80 type, because a string of 80 was considered enough for main uses because is sufficient to fill a row in the old 80 columns monitors. It was not normal to use the 255 chars, because it was considered an unnecessary memory  spending.
    
When you made the  operation:
    
    s2:=s1; 
//  the entire block of memory of the s1 string var was copied in the memory block allocated by s2. So it was an operation to prevent if you can, because move a memory block was a slow operation.
    
There is several web pages than talk about the string history. you can read it for curiosity. 

The fact is: the string was changed in pascal history. The optimal code in Tp versions can not work in delphi in the correct way. Because it was normal to handle the s[0] position to handle string length, and now it will be an runtime error. Even in 0 based strings like if you explicit declare it, or compile for android mode, the s[0], is now the first string char, not the length.
    
    Is very easy to use strings in delphi (  compared with classic c).  Concatenate string by using + operator is easy code  to write and read.

Much of the code used to compose reports , make information to display or write xml responses, was repetitive string operation, coping from a part of ones to others which concatenate  several strings or substrings.

The Delphi compiler makes  it easy, but translated to machine code, the string operation had a overload cost  because it creates temporary string variables in some operation, which uses lot of allocate-free memory operation.

for i:=0 to 1000 do 
begin
  s:=s+’x’
end;

It produces at least 1000 calls to memory reallocation management. In a more complex loop, with several string modifications, the calls to memory management grown a lot.

    It sound  like a joke, some time we don’t want to use classes, because it need memory allocation, creation call, and destroy, And we haven’t problem to use strings, forgetting that an string is similar to an object or class. Alloc memory and releases it when finish the use.

It’s true that use a string is easier than a class, because we forget the constructor and destructor, it was created automatic by the compiler (Like the variants)

Then is better to think in the string like it was an Object or class, but it is a special class; it is reference-counted. 

    o1:=TObjeto.Create;  //  TObjeto instance was created. o1 is a pointer to this instance.    o2:=o1 // the o2 pointer, point to the same data than o1.
o1.Free ; //free the  o1 object instance.
o2.free;//  memory ERROR, o2 point to a released block , 

Lets to see what happen if it was String;

s1:=’hello world’; /// a block of memory was allocated to hold the char information.
s2:=s1;  // It means than s2 point to the same memory block than s1.it didn’t take a new memory block, 
s1:=’’’; // s1 is now null, but no memory was released, because s2 point to it too..
s2:=’’; // put  nil the s2 string, and free the memory allocated by s1, in the first line.


This  feature can be a problem if you change  the chars of the strings directly, because you can corrupt other strings that reference the same memory block.

In the example,only one allocation of memory operation was done in the first line, and only a free operation in last. It not duplicates the memory blocks or move memory between the s1 and s2 vars.


¿  How can we use it for memory optimization ?

    There is several text structures with lot of repeated substrings.
(for example XML, or Json)

Look at the next table:

<tabla1>
<RG nombre=”fulano”>
<RG nombre=”mengano”>
<RG nombre=”sotano”>
</tabla1>

An XML parse-reader must to read one node with name tabla1, and 3 child nodes with the name of RG. It mean than 3 xml nodes (or objects) hold the same name information, allocated in a string in different memory blocks, if an intelligent reader uses the same memory to be pointed for the 3 strings , then we don’t need the other 2 blocks.

Normally an xml parser,  create new strings containing the document blocks that was reading. If we have the way to search if a string was in memory, then we don’t need to create, it again, we only need to use the point to the  string jet in memory

The challenge is : if we have a string container with the used strings.
 When new string was generated in the parser , first, you search the string in the container. If the string is not in the container jet, then include the string in ( so next instances to the same string don’t need to be allocated.) If the newstring was in the container, then we take the string of the container and the generated newstring was released.. 

The component is not a dictionary in the strict way, because it isn’t a dupla, or a string label which point or index to an object (or other string).
Really we only need  a string list which represent nothing, only to themselves.
Is not a dictionary and is more similar to a simple array of strings, so I call it 
Strinary, 
(sorry if the world is used for other purposes, but AFAIK it hasn’t)


There is some components which help in global solutions by using specialized memory managements with multi-user optimization, but really i didn’t find special components which explode this feature. I Hope that this “article” open your curiosity and we all find the better way to work with the Strinary .At the moment I want to share my own experience.

To add “Strinary” to any existing code, can be complicated, even dangerous.
If the list of different string is big , there is a lost of time for searching strings blocks.
So the ideal use of “Strinary” is in encapsulated process with no so much different strings, but with lot of each one of them. That is XML.

When there is a lot of different strings, then it reduces the possibility to find the same string, so not a lot of memory reduction was possible, and spend more time in searching if the new string is or not in the Strinary container. It hasn’t magical solution.

We need to search a good relation of memory/time consumption  

A lot of memory saving , and just spend a little time more, is Ideal.
And  few memory saving and lot of time spended is the opposite. 
But we can have cases with XML files with 300 or 400 MB, sometimes, there is not possible to read this in memory even if we spend the 2gb of the 32bits  application.

This is the ideal stage for  “Strinary”, when we need to read a big file, the space saved can justify the spend of the time. Even more, in the real use, we can find little surprises, and it depending of the size of the strings and the number, it can be that you save lot of space, and even it was faster.

An sceptic reader can think that this is murky issues, or demagogy. Maybe have the reason to suspect. I am still surprised with some experiment, and I am the first to suspect with the result. So I will share the code to can be contrasted with other colleagues.

My intention is  to document the way to do the same operations with big xml files in different XML component libraries. Looking for the bad and good points of each one, and trying to optimize the memory consumption by using Strinary.

So you (and me)  will have factual information about this libraries to choose the better library for your purpose.

To make an Strinary component is easy, The difficult is to do it more and more quick, and flexible to each purpose. 

Any delphi programmer knows the TStringList class, and the sorted property and unique property. The TStrinary component is like a string list (sorted for faster search), and unique string in each item.
    
    One of the curious of TStrinary is that can be destroyed at end of parsing, without affect to the created xml objects, because the strings in the container will have more than 1 reference-count, so will not be released until the xml objects are destroyed.
    
    So TStrinary  must not have side effects, only a spend of time in parsing, but then will be work at the same speed than the original way (without strinary)

Now, I will compare 4 different components or libraries to work with xml or Json. To check the performance with big files (at least 30 mb), and huge files (300mb or more).

Json in Mormot, OXml, Frxxml ( FastScript / Fastreport) and  NativeXml.

I have been disappointed with  NativeXml, it is slow , and spend lot of memory.

In  OXML , it generates (or can generate) DOM compatible interfaces, so they have an additional overload for this feature. Really is much more faster than nativexml, but spend a little more of memory (at least with the sample xml that i used)


    The surprise was  TfrxXMLDocument, , it uses a really simple parsed, and it uses a trick to win space and time, it don’t parses the xml attributes or generate a list of attributes like other does, It copies the entire string with all the attributes in the node  as a  single string.
After, when it is needed to access to one property, it search with a simple text search function in the all attributes string and take the value into the  “ “ .
     Really it not validates the XML, so is the more light in code,

    It is not MS-DOM compatible (like OXML can), but searching is simple, and can be effective for the main uses.
    The only big problem is when deleting the TfrxXMLDocument, the destroy of the nodes is slow, because it was deleting from the owner children list , each released object. one by one, searching in the child node  list before deleted.
    When you deleted all the nodes, the result is that the owner child list will be finally released. So is better destroy the nodes without delete it from the list,  and at final, clear the entire list,

Because it  is the easier,  it was easy to add strinay to use it with the frx node names .


 exaple1. 37mb xml file) it saved the 35% of memory, without difference in speed (less than 1% of time difference)  It is a good component for use Strinary.


OXML, has their own dictionary, really is a sophisticated hash system and it could do fast search.  Is difficult to add Strinary to OXml, But ...maybe mixing with a hash system it could be possible even a faster way to do Strinay. (is a To Do)

Recently, I’m experimented with mORMot. I like the Json power, because is faster than others XML, but it spend lot of memory, even more than Oxml, so it’s difficult to use for our purposes. Even more, because  a special feature for handling the temporal string in parsing (putting a #0 char at the end of string) is needed to copy all the entire string to parse. It do it more difficult to parse large files.
Because it spènd a lot of memory , it is a good candidate for Strinary.

The result is that we can read the Json, in less of the middle of the normal size. But the cost in time is the double too.
It depending of the priority can be useful in some cases. More if you consider that originally is very fast.
The question you must to do is: I want to read a file in a 10 millisecond , and it takes 2Mb of Ram, or I prefer to read the same file in 20 millisecon but using only 1 Mb in RAM.?

    The response it depend of the project , some time we spend lot of memory and code to win some millisecond, and other time we spend a lot of time in compress some information. You can experimented with the libraries to know what is the better for your purpose.

The Strinary is simple. I wonder is no longer used. Or there was some native specific delphi component. 

My response to the first question is: By using this “hide” feature of the string, it could be possible  to read an entire file in memory spending less space that original file size. Of course  only in  some special files with longs and repetitive tokens.

    
        
