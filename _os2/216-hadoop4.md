---
layout: lekcija
title: Spajanje
main_category: Materijali za vežbe
sub_category: Apache Hadoop
image: join.png
active: false
comment: true
---

Neka su dati fajlovi koji sadže podatke o kupcima u sledećem obliku:

ID_kupca, Ime, Broj_Telefona

i neka su dati podaci o narudžbenicama u sledećem obliku:

ID_kupca, ID, Naziv_proizvoda, Datum

Potrebno je kreirati **map/reduce** programski model koji ispisuje, na standardnom izlazu, koji je kupac kreirao svaku narudžbenicu. Izlaz je oblika:

ID_kupca, Ime, Broj_Telefona, ID_kupca, ID, Naziv_proizvoda, Datum

**Main** metod kao argumente prihvata putanje do podataka o kupcima, narudžbenicama i izlaznih fajlova.

# Predlog rešenja

JoinMapperKupci.java
{% highlight java %}


import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class JoinMapperKupci extends Mapper<LongWritable, Text, Text, Text>
{
    @Override
    public void map(LongWritable key, Text linija, Context context)
            throws IOException, InterruptedException
    {
        String linijaTmp = linija.toString();

        String[] vrednosti = linijaTmp.split(",");

        Text izlazniKljuc = new Text();
        izlazniKljuc.set(vrednosti[0]);

        Text izlaznaVrednost = new Text();
        izlaznaVrednost.set("kupci," + linijaTmp);

        context.write(izlazniKljuc,izlaznaVrednost);
    }
}
{% endhighlight %}

JoinMapperNarudzbenice.java
{% highlight java %}

import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class JoinMapperNarudzbenice extends Mapper<LongWritable, Text, Text, Text>
{
    @Override
    protected void map(LongWritable kljuc, Text linija, Context context) throws IOException, InterruptedException
    {
        String linijaTmp = linija.toString();

        String[] vrednosti = linijaTmp.split(",");

        Text izlazniKljuc = new Text();
        izlazniKljuc.set(vrednosti[0]);

        Text izlaznaVrednost = new Text();
        izlaznaVrednost.set("narudzenica," + linijaTmp);

        context.write(izlazniKljuc,izlaznaVrednost);
    }
}
{% endhighlight %}

JoinReducer.java
{% highlight java %}

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class JoinReducer extends Reducer<Text, Text, Text, Text>
{

    /**
     *
     * @param kljuc
     * @param vrednosti
     * @param context
     * @throws InterruptedException
     * @throws IOException
     */
    @Override
    protected void reduce(Text kljuc, Iterable<Text> vrednosti, Context context) throws java.lang.InterruptedException, IOException
    {
        Map<String, List<String>> mapa = new HashMap<String, List<String>>();

        //Kreiraj mapu za kupce
        mapa.put("kupci", new ArrayList<String>());
        //Kreiraj mapu za narudzbenice
        mapa.put("narudzenica", new ArrayList<String>());

        for(Text val : vrednosti)
        {
            String trenutnaVrednost = val.toString();

            //Uzmi ime tabele od koje potice vrednost
            String imeTabele = trenutnaVrednost.split(",")[0];

            //Uzmi mapu izvorne tabele
            List<String> tabela = mapa.get(imeTabele);

            //Izbrisi ime tabele od koje potice podatak
            String zamenjenString = trenutnaVrednost.replace(imeTabele + ",", "");

            //Dodaj vrednost u odgovarajucu mapu tabele
            tabela.add(zamenjenString);

        }


        List<String> kupci = mapa.get("kupci");
        List<String> narudzenice = mapa.get("narudzenica");

        //Proveri da li je kupac nesto narucio
        if(narudzenice != null)
        {
            for(String red1 : kupci)
                for(String red2 : narudzenice)
                {
                    Text vrednost = new Text();
                    vrednost.set(red1 + "," + red2);
                    context.write(new Text(),vrednost);
                }
        }

    }
}
{% endhighlight %}

JoinDriver.java
{% highlight java %}


import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.logging.Level;
import java.util.logging.Logger;
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

public class JoinDriver extends Configured implements Tool
{
    public static void main(String[] args)
    {
        try
        {
            ToolRunner.run(new JoinDriver(), args);
        }
        catch (Exception ex)
        {
            Logger.getLogger(JoinDriver.class.getName()).log(Level.SEVERE, null, ex);
        }
    }

    public int run(String[] strings) throws Exception
    {
        Path outputPath = new Path(strings[2]);

        //Izbrisi sve fajlove iz foldera u kome ce se naci rezultat
        FileSystem fs = FileSystem.get(new Configuration());
        fs.delete(outputPath, true);

        Job job = new Job(getConf());
        job.setJobName("Nas prvi join");

        //Setuj klasu kojoj pripadaju izlazni kljuc i izlazna vrednost mapera
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        //Setuj klasu kojoj pripadaju izlazni kljuc i izlazna vrednost
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        //Setuj klasu koja sadrze definiciju reduce-ra
        job.setReducerClass(JoinReducer.class);

        //Setuj map-ere
        MultipleInputs.addInputPath(job, new Path(strings[0]), TextInputFormat.class, JoinMapperKupci.class);
        MultipleInputs.addInputPath(job, new Path(strings[1]), TextInputFormat.class, JoinMapperNarudzbenice.class);

        //Setuj putanju za izlazne fajlove
        FileOutputFormat.setOutputPath(job, new Path(strings[2]));

        //Setuj klasu koja sadrzi main funkciju
        job.setJarByClass(JoinDriver.class);

        //Sacekaj dok se posao ne zavrsi
        boolean success = job.waitForCompletion(true);


        //Ako je posao uspesan
        if(success)
        {
            System.out.println("Posao je uspesno izvrsen!");

            Path filePath = new Path(strings[2] + "/part-r-00000");

            BufferedReader br = new BufferedReader(new InputStreamReader(fs.open(filePath)));

            String line = br.readLine();

            //Stampaj sadrzaj izlaznog fajla
            while(line != null)
            {
                System.out.println(line);
                line = br.readLine();
            }
        }
        else
            System.out.println("Posao nije uzpesno izvrsen!");

        return 0;
    }

}
{% endhighlight %}
