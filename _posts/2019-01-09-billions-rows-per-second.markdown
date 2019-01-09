---
layout: post
title:  "A Billion Rows A Second"
date:   2019-01-09 12:00:00
categories: [Data Science, Big Data]
tags: [Data Science, Big Data]
---

Have you ever started a new project, got the raw dataset and realized the data was way bigger than you could possibly deal with on your machine? Have you ever wrote a script in Python that runs to the point of taking an infinite number of days?

Both of these problems have solutions, but none are easy to implement for those without a focus on big-data. Until now.

![
](https://images.unsplash.com/photo-1515348152804-f0a800ce7f7b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

I recently ran across a new library called [Vaex](https://github.com/vaexio/vaex/blob/master/docs/source/index.ipynb), and in short -it's incredibly useful. Vaex author Maarten Breddels describes it perfectly in his [introductory post](https://towardsdatascience.com/vaex-out-of-core-dataframes-for-python-and-fast-visualization-12c102db044a):

>Wouldn’t it be great if you could load a 1 Terabyte data file instantly, reading only the parts you need, with a strategy that smart kernel programmers have spent decades optimizing?

> And while making insane requests, why don’t we ask for an API that feels a bit like pandas, that we all use. Oh yeah, and please don’t use too much memory on my 2013 MacBook Air while you’re at it. Since it’s 2018 and we all work in the Jupyter notebook, make it work with that as well will you? Make sure that all of my open notebooks also share the memory of that dataset as well please.

### Taming the Beast
For those of us without a decade of experience working in the Hadoop ecosystem (or those of us who have …ahem… had software engineers “helping” with the hard part), dealing with massive datasets is extremely painful. Vaex doesn’t exactly work out of the box if you are used to working with Pandas, so I want to lower the barrier to entry here.

### DataTypes
First, we need to talk about datatypes. Under the hood Vaex wraps a set of numpy arrays. Most datasets come in the form of a CSV file, and that’s what many data scientists will be used to working with. But, have you ever tried to read a 20gb file into memory? For most computers, it’s an impossibility.

So we want to convert our files into a format that Vaex loves (HDF5) rather than having Vaex convert via Pandas. The typical case for large CSV files is to have them broken up into part-files, so I wanted to write a bash script that could handle both cases.

For our example file, we’re going to use a [Kaggle dataset on air quality](https://www.kaggle.com/epa/hazardous-air-pollutants) (~2.5GB unzipped), and we’ll remove most of the string columns. That leaves us with a ~1GB file. Small as far as data goes, but good for this demo.


 1. To start, you need to download a handy tool for file conversion called [Topcat](http://www.star.bris.ac.uk/~mbt/topcat/#standalone). If you don't have experience using .jar files, you can read about them [here](https://en.wikipedia.org/wiki/JAR_(file_format)).

 2. Copy the jar into the directory of the CSV file (you could also just have a central directory of jar files but I'll leave that to the reader).

 3. Now we use a custom bash script to convert the CSV efficiently to HDF5

```bash
#!/bin/bash

#Variables
csv_files='ls /path/to/csv/files/*.csv'
out_file='/path/to/output/folder'
file_locs=$out_file/files.txt
fits_out=$out_file/merged.fits
hdf5_out=$out_file/merged.hdf5

# Create Txt File of Locations
> $file_locs
for f in $csv_files
do
    if [ $f = 'ls' ]; then
        continue
    fi
    echo "$f" >> $file_locs
done

#Create .fits Files and Convert to hdf5
echo 'Compiling Fits Files For Conversion to HDF5'
./topcat_jar/topcat -stilts -Djava.io.tmpdir=/tmp tcat in=@$file_locs ifmt=csv out=$fits_out ofmt=colfits

#Use Vaex to convert .fits to HDF5
echo 'Converting .fits to HDF5'
source  /path/to/venv # if applicable
vaex convert file $fits_out $hdf5_out
rm $fits_out
rm $file_locs
echo 'Process Complete: Deleting .txt file and .fits intermediary file'
echo 'HDF5 saved to: $hdf5_out'
```

And we are ready to test out Vaex. Since Vaex doesn't actually load the dataset into memory, it is almost instantaneous.
![
](https://i.imgur.com/Noxlwg4.png)

Let's run a relatively simple calculation and compare. Our dataset contains Longitudes and Latitudes for each datapoint (there are a total of ~ 9 Million rows. Let's say we want to find all points contained within New York City.

A rough box might give one the following:
nyc_lat = [40.348424, 40.913145]
nyc_long = [-74.432885, -73.696834]

And here is the comparison of Vaex vs Pandas.
![
](https://i.imgur.com/M61AqH9.png)
For this simple operation, Vaex is almost 25 times fast. Keep in mind that the larger your dataset the more benefit you are going to see from Vaex.

Additionally, complex mathematical operations can be significantly boosted using a beautiful Python library called Numba (more on that on a later post).

### Powerful Data Science Tools
I always find that big-data creates its own unique set of challenges. Datasets that number in the Terabytes, mathematical ops numbering in the trillions, and the complexities that both bring make life significantly more complicated.

Libraries like Vaex can allow data scientists without the deep computing expertise that a big-data engineer has and still able to power through incredibly large datasets in a very efficient way.

If you’re interested in learning more about handling big-data in a small way (i.e. by yourself), then follow me on Medium, or get in touch at https://jessemoore.ca to discuss a potential project.

Till next time...

Jesse
