---
layout: lekcija
title: Indekser linija
main_category: Materijali za vežbe
sub_category: Apache Hadoop
image: indexer.png
active: false
comment: true
---

Neka su dati fajlovi u kojima se nalazi proizvoljni tekst. Potrebno je kreirati **map/reduce** programski model koji za svaku reč u tekstu ispisuje u kojim se fajlovima nalazi. **Main** metod kao argumente prihvata putanje do ulaznih fajlova, izlaznih fajlova i broj reduktora.

Primer:
{% highlight bash %}
f1.txt
Pera ide u školu.
Pera hoda.
{% endhighlight %}
{% highlight bash %}
f2.txt
Pera ne dolazi na OS2.
{% endhighlight %}
{% highlight bash %}
izlaz.txt
Pera f1.txt, f1.txt, f2.txt
id f1.txt
u f1.txt
školu. f1.txt
ne f2.txt
dolazi f2.txt
na f2.txt
OS2. f2.txt
{% endhighlight %}

## Predlog rešenja

{% highlight java %}
import java.io.IOException;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;



public class LineIndexer3 extends Configured implements Tool
{
  public static class LineIndexMapper extends Mapper<LongWritable, Text, Text, Text>
  {
    private final static Text word = new Text();
    private final static Text location = new Text();

    public void map(LongWritable key, Text value, Context context) throws java.lang.InterruptedException, IOException
    {

      FileSplit fileSplit = (FileSplit)context.getInputSplit();
      String fileName = fileSplit.getPath().getName();
      //location.set(fileName);

      String line = value.toString();

      String[] words = line.split(" ");

      for(int i = 1; i < words.length; i++)
      {
        word.set(words[i]);
        location.set("(" + fileName + ", " + words[0] + ", " + i + ")");
        context.write(word, location);
      }
    }
  }

  public static class LineIndexReducer extends  Reducer<Text, Text, Text, Text>
  {

    public void reduce(Text key, Iterable<Text> values, Context context)
    throws java.lang.InterruptedException, IOException
    {
      boolean first = true;
      String toReturn = "";
      for (Text val : values)
      {
        if (!first)
          toReturn += ", ";

        first=false;
        toReturn += val.toString();
      }

      context.write(key, new Text(toReturn));
    }
  }

  public int run(String[] args) throws Exception
  {
    Path pt=new Path(args[1]);
    FileSystem fs = FileSystem.get(new Configuration());
    //delete recursive
    fs.delete(pt, true);

    int numReduceTasks = Integer.parseInt(args[2]);

    Job job = new Job(getConf());

    job.setJarByClass(LineIndexer3.class);
    job.setJobName("Indexer");

    job.setMapOutputKeyClass(Text.class);
    job.setMapOutputValueClass(Text.class);

    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(Text.class);

    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));

    job.setMapperClass(LineIndexMapper.class);
    job.setReducerClass(LineIndexReducer.class);

    job.setNumReduceTasks(numReduceTasks);

    boolean success = false;
    success = job.waitForCompletion(true);


    if(success)
    {
      System.out.println("Posao je zavrsen uzpesno\n");

      for(int i = 0; i < numReduceTasks; i++)
      {
        String fileName = "part-r-0000" + i;
        System.out.println(fileName);
        pt = new Path(args[1] + "/" + fileName);
        fs = FileSystem.get(getConf());
        try
        {
          BufferedReader br = new BufferedReader(new InputStreamReader(fs.open(pt)));
          String line = br.readLine();
          while(line != null)
          {					
            System.out.println(line);
            line = br.readLine();
          }
        }
        catch(IOException ioe)
        {
          System.out.println(ioe);
        }
      }


    }
    else
      System.out.println("Posao nije zavrsen uzpesno\n");

    return success ? 1 : 0;
  }

  public static void main(String[] args)
  {
    try
    {
      System.exit(ToolRunner.run(new LineIndexer3(), args));
    }
    catch (Exception e)
    {
      System.out.println(e);
    }
  }
}
{% endhighlight %}

# Domaći

Prilagoditi prethodno rešenje tako da ispisuje u kom fajlu se nalazi i koliko se puta javlja odrđena reč.

Primer:
{% highlight bash %}
f1.txt
Pera ide u školu.
Pera hoda.
{% endhighlight %}
{% highlight bash %}
f2.txt
Pera ne dolazi na OS2.
{% endhighlight %}
{% highlight bash %}
izlaz.txt
Pera f1.txt 2, f2.txt 1
id f1.txt 1
u f1.txt 1
školu. f1.txt 1
ne f2.txt 1
dolazi f2.txt 1
na f2.txt 1
OS2. f2.txt 1
{% endhighlight %}
