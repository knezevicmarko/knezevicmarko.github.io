---
layout: lekcija
title: Like
main_category: Materijali za vežbe
sub_category: Apache Hadoop
image: like.png
active: true
comment: true
---

Neka su date strukture fajlova:

* KORISNIK (idKorisnika, ime)
* PRIJATELJI ( idKorisnika1, idKorisnika2)
* POST ( idPosta, idKorisnika, sadržaj)
* LIKE ( idKorisnika, idVlasnikaPosta, idPosta)

Potrebno je pronaći imena korisnika koji su lajkovali barem jedan post svakom svom prijatelju. Programsko rešenje treba da sadrži odgovarajuće klase sa konfiguracijom za pokretanje posla na Hadoop-u. Imena traženih korisnika je potrebno štampati na standardnom izlazu.

# Predlog rešenja

MapperKorisnici.java
{% highlight java %}

import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;


public class MapperKorisnici extends Mapper<LongWritable, Text, Text, Text>
{
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String vrednost = value.toString();
        String[] vrednosti = vrednost.split(",");
        Text kljuc = new Text();
        Text v = new Text();
        kljuc.set(vrednosti[0]);
        v.set("K," + vrednosti[1]);
        context.write(kljuc, v);
    }
}
{% endhighlight %}

MapperLike.java
{% highlight java %}
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class MapperLike extends Mapper<LongWritable, Text, Text, Text>
{
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String vrednost = value.toString();
        String[] vrednosti = vrednost.split(",");
        Text kljuc = new Text();
        Text v = new Text();
        kljuc.set(vrednosti[0]);
        v.set("L," + vrednosti[1]);
        context.write(kljuc, v);
    }
}
{% endhighlight %}

MapperPrijatelji.java
{% highlight java %}
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;


public class MapperPrijatelji extends Mapper<LongWritable, Text, Text, Text>
{
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String vrednost = value.toString();
        String[] vrednosti = vrednost.split(",");
        Text kljuc1 = new Text();
        Text kljuc2 = new Text();
        Text v1 = new Text();
        Text v2 = new Text();
        kljuc1.set(vrednosti[0]);
        v1.set("P," + vrednosti[1]);
        kljuc2.set(vrednosti[1]);
        v2.set("P," + vrednosti[0]);
        context.write(kljuc1, v1);
        context.write(kljuc2, v2);
    }
}
{% endhighlight %}
ReducerKorisnici.java
{% highlight java %}
import java.io.IOException;
import java.util.HashSet;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class ReducerKorisnici extends Reducer<Text, Text, Text, Text>
{
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException
    {
        HashSet<String> prijatelji = new HashSet<String>();
        HashSet<String> lajkovi = new HashSet<String>();
        String ime = new String();
        for (Text v : values)
        {
            String vrednost = v.toString();
            String[] v1 = vrednost.split(",");
            if (v1[0].equals("K"))
            {
                ime = v1[1];
                continue;
            }
            if (v1[0].equals("P"))
            {
                prijatelji.add(v1[1]);
                continue;
            }
            lajkovi.add(v1[1]);
        }

        if (prijatelji.size() == lajkovi.size())
        {
            context.write(new Text(), new Text(ime));
        }
    }
}
{% endhighlight %}
Driver.java
{% highlight java %}
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class Driver extends Configured implements Tool
{
    public static void main(String[] args)
    {
        try
        {
            ToolRunner.run((Tool)new Driver(), (String[])args);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }

    public int run(String[] arg0) throws Exception
    {
        Path outputPath = new Path(arg0[3]);
        FileSystem fs = FileSystem.get((Configuration)new Configuration());
        fs.delete(outputPath, true);
        Job job = new Job(this.getConf());
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setReducerClass(ReducerKorisnici.class);
        MultipleInputs.addInputPath((Job)job, (Path)new Path(arg0[0]), TextInputFormat.class, MapperKorisnici.class);
        MultipleInputs.addInputPath((Job)job, (Path)new Path(arg0[1]), TextInputFormat.class, MapperPrijatelji.class);
        MultipleInputs.addInputPath((Job)job, (Path)new Path(arg0[2]), TextInputFormat.class, MapperLike.class);
        FileOutputFormat.setOutputPath((Job)job, (Path)outputPath);
        job.setJarByClass(Driver.class);
        boolean rezultat = job.waitForCompletion(true);
        if (rezultat)
        {
            Path izlaz = new Path(String.valueOf(arg0[3]) + "/part-r-00000");
            BufferedReader br = new BufferedReader(new InputStreamReader((InputStream)fs.open(izlaz)));
            String linija = br.readLine();
            while (linija != null)
            {
                System.out.println(linija);
                linija = br.readLine();
            }
        }
        else
        {
            System.out.println("Posao nije uspesno zavrsen.");
        }
        return 0;
    }
}
{% endhighlight %}
