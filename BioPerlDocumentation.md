Hybrid Species -- Documentation
==============================================

The following documentation was written with Mac OS X and UNIX in mind. I encourage you to unzip the Hybrid Species Research.zip file into your `/Documents` folder as this tutorial will work based on storing the files there (ex `/Documents/Hybrid Species Research`).

The first thing we want to do is install the latest version of perl. To do this we will use Perlbrew to do the installation:

```bash
$ \curl -L http://install.perlbrew.pl | bash
```

Once Perlbrew is downloaded and installed, you need to install perl. The latest version is 5.20.1 (as of December 5, 2014). Check perl's website for the latest version:

```bash
$ perlbrew install perl-5.20.1
$ perlbrew switch perl-5.20.1
```




##Installing BioPerl Apps

To do all of the installation, we will be using CPAN (Comprehensive Perl Archive Network). First, let's make sure that CPAN is installed and up to date:

```bash
$ install CPAN
$ reload CPAN
```

Then we can install the necessary applications with the following (the install could take a few minutes):

```bash
$ sudo cpan
cpan[1]> install Bio::Tree:Tree

Do you want to run the Bio::DB::GFF or Bio::DB::SeqFeature::Store live database tests? y/n **[n]** 

Install [a]ll BioPerl scripts, [n]one, or choose groups [i]nteractively? **[a]**

Do you want to run tests that require connection to servers across the internet
(likely to cause some failures)? y/n **[n]**


cpan[2]> install Bio::Tree::Node

cpan[3]> install Bio::Tools::Run::Alignment::Clustalw

Install scripts? y/n **[n]**

Do you want to run tests that require connection to servers across the internet
(likely to cause some failures)? y/n **[n]**

**THIS WILL THROW A FAILURE** RUN THIS:

cpan[4]> force install CJFIELDS/BioPerl-Run-1.006900.tar.gz



cpan[5]> install Bio::SeqIO

Do you want to run the Bio::DB::GFF or Bio::DB::SeqFeature::Store live database tests? y/n **[n]**

Do you want to run tests that require connection to servers across the internet
(likely to cause some failures)? y/n **[n]**

cpan[6]> install Bio::TreeIO
```

Now that we have all the dependencies installed, we can install clustalw, which is what we use to perform the sequence alignment algorithms. 


##Using Clustalw 

Included is a Clustalw executable located in `/Documents/Hybrid Species Research/bin/clustalw-2.1-macosx`. The file is called `clustalw2`. Our program looks for the executable named `clustalw`. 

Rename `clustalw2` to `clustalw`. You can right click and select rename to perform this action or you can do it at the command line after navigating to the Hybrid Species Research folder:

```bash
$ cd bin/clustalw-2.1-macosx
$ mv clustalw2 clustalw
```

The tricky part now is to add `clustalw` to the global `$PATH`. We do this so that clustalw can be run by just typing `clustalw` from the command line. To do this we will do move the `clustalw` executable to somewhere that is easy for the computer to access it (for some reason, it does not work if the path has a space in it). 

```bash
$ cd /Users/YourUsername/
$ mkdir clustalw
```

Now that you have a directory set up in your home, copy the `clustalw` executable from `/Documents/Hybrid Species Research/bin/clustalw-2.1-macosx` and place it into `/Users/YourUsername/clustalw`. You can also do this from the command line once inside the new clustalw folder:

```bash
$ cp Documents/Hybrid\ Species\ Research/bin/clustalw-2.1-macosx/clustalw .
```

This command copies the executable from the original location into the current directory (.) which should be `/Users/YourUsername/clustalw`.

Now lets add `clustalw` to your path. In the following, I use Nano to edit. Use whatever text editor you like.

```bash
$ sudo nano ~/.bash_profile
```

Add the following line and then hit `ctrl + x` to exit. Save the changes by hitting `y`. Finally hit `enter` to save.

```
export PATH=$PATH:~/clustalw/
```

No you need to close the current terminal window and open up a new one in order to refresh and save all the changes. Type the following to ensure that `clustalw` was added to your path:

```bash
$clustalw




 **************************************************************
 ******** CLUSTAL 2.1 Multiple Sequence Alignments  ********
 **************************************************************


     1. Sequence Input From Disc
     2. Multiple Alignments
     3. Profile / Structure Alignments
     4. Phylogenetic trees

     S. Execute a system command
     H. HELP
     X. EXIT (leave program)


Your choice: 
```

Cool! We are in!! Hit exit to leave the clustalw program. You will not need to interact with this program as BioPerl will do it for you.


##Run the Program

Inside the directory for `Hybrid Species Research` run the following to execute the program:

```bash
perl alignment_based.pl
```




