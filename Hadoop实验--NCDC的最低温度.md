# Hadoop实验——美国德克赛斯州的2015年所有的最低气温（2015年）

## 1.实验要求：

1）根据一天的气温对每天进行冷热分类，并求“cold day”的最低气温。

2）求2015年的最低气温。

## 2.下载实验的数据集：

1.网址：[ftp://ftp.ncdc.noaa.gov/pub/data/uscrn/products/daily01/2015/CRND0103-2015-TX_Austin_33_NW.txt](ftp://ftp.ncdc.noaa.gov/pub/data/uscrn/products/daily01/2015/CRND0103-2015-TX_Austin_33_NW.txt)

下载方式推荐使用软件：Filezilla，这个软件访问下载ftp的资源很方便。

2.数据集说明：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221110202308906.png" alt="image-20221110202308906" style="zoom:200%;" />



<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221110202448524.png" alt="image-20221110202448524" style="zoom:150%;" />



<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221110202721618.png" alt="image-20221110202721618" style="zoom:150%;" />

## 3.编写代码实现MapReduce的过程：

编写代码实现将数据集中的T_DAILY_MAX	,T_DAILY_MIN这 两个字段的数据提出来根据温度高低做天气的分类。通过MapReduce实现大数据处理的过程。

代码如下：

    import java.io.IOException;
    import java.util.Iterator;
    
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
    import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.conf.Configuration;
    
    public class MyMaxMin {
    
    
            //Mapper
    
            /**
            *MaxTemperatureMapper class is static and extends Mapper abstract class
            having four hadoop generics type LongWritable, Text, Text, Text.
            */
    
            public static class MaxTemperatureMapper extends
                            Mapper<LongWritable, Text, Text, Text> {
    
                    /**
                    * @method map
                    * This method takes the input as text data type.
                    * Now leaving the first five tokens,it takes 6th token is taken as temp_max and
                    * 7th token is taken as temp_min. Now temp_max > 35 and temp_min < 10 are passed to the reducer.
                    */
    
                    @Override
                    public void map(LongWritable arg0, Text Value, Context context)
                                    throws IOException, InterruptedException {
    
                    //Converting the record (single line) to String and storing it in a String variable line
    
                            String line = Value.toString();
    
                    //Checking if the line is not empty
    
                            if (!(line.length() == 0)) {
    
                                    //date
    
                                    String date = line.substring(6, 14);
    
                                    //maximum temperature
    
                                    float temp_Max = Float
                                                    .parseFloat(line.substring(39, 45).trim());
    
                                    //minimum temperature
    
                                    float temp_Min = Float
                                                    .parseFloat(line.substring(47, 53).trim());
    
                            //if maximum temperature is greater than 35 , its a hot day
    
                                    if (temp_Max > 35.0) {
                                            // Hot day
                                            context.write(new Text("Hot Day " + date),
                                                            new Text(String.valueOf(temp_Max)));
                                    }
    
                                    //if minimum temperature is less than 10 , its a cold day
    
    								if (temp_Min < 10) {
                                            // Cold day
                                            context.write(new Text("MinCold Day " + date),
                                                            new Text(String.valueOf(temp_Min)));
                                    }
                            }
                    }
    
            }
    
    //Reducer
    
            /**
            *MaxTemperatureReducer class is static and extends Reducer abstract class
            having four hadoop generics type Text, Text, Text, Text.
            */
    
            public static class MaxTemperatureReducer extends
                            Reducer<Text, Text, Text, Text> {
    
                    /**
                    * @method reduce
                    * This method takes the input as key and list of values pair from mapper, it does aggregation
                    * based on keys and produces the final context.
                    */
    
                    public void reduce(Text Key, Iterator<Text> Values, Context context)
                                    throws IOException, InterruptedException {
    
    
                            //putting all the values in temperature variable of type String
    
                            String temperature = Values.next().toString();
                            context.write(Key, new Text(temperature));
                    }
    
            }
    
    
    
            /**
            * @method main
            * This method is used for setting all the configuration properties.
            * It acts as a driver for map reduce code.
            */
    
      public static void main(String[] args) throws Exception {
    
                          //reads the default configuration of cluster from the configuration xml files
                Configuration conf = new Configuration();
    
                //Initializing the job with the default configuration of the cluster
                Job job = new Job(conf, "weather example");
    
                //Assigning the driver class name
                job.setJarByClass(MyMaxMin.class);
    
                //Key type coming out of mapper
                job.setMapOutputKeyClass(Text.class);
    
                //value type coming out of mapper
                job.setMapOutputValueClass(Text.class);
    
                //Defining the mapper class name
                job.setMapperClass(MaxTemperatureMapper.class);
    
                //Defining the reducer class name
                job.setReducerClass(MaxTemperatureReducer.class);
    
                //Defining input Format class which is responsible to parse the dataset into a key value pair
                job.setInputFormatClass(TextInputFormat.class);
    
                //Defining output Format class which is responsible to parse the dataset into a key value pair
                job.setOutputFormatClass(TextOutputFormat.class);
    
                //setting the second argument as a path in a path variable
                Path OutputPath = new Path(args[1]);
    
                //Configuring the input path from the filesystem into the job
                FileInputFormat.addInputPath(job, new Path(args[0]));
    
                //Configuring the output path from the filesystem into the job
                FileOutputFormat.setOutputPath(job, new Path(args[1]));
    
                //deleting the context path automatically from hdfs so that we don't have delete it explicitly
                OutputPath.getFileSystem(conf).delete(OutputPath);
    
                //exiting the job only if the flag value becomes false
                System.exit(job.waitForCompletion(true) ? 0 : 1);
    
        }
    }
## 4.编译文件计算结果：

1.编译程序MyMaxMin.java（文件名要和类名一样，不然会报错）。

命令：

```
##定义 HADOOP_CLASSPATH
export HADOOP_CLASSPATH=$(hadoop classpath)
echo HADOOP-CLASSPATH
##编译Java
javac -classpath ${HADOOP_CLASSPATH} -d climate_class/ MyMaxMin.java 
```

2.生成jar包（只有jar包才能被Hadoop使用）

命令：

```
jar -cvf MyMaxMin.jar -C climate_class/ .##生成.jar文件
```

3.上传数据到HDFS节点：

```
Hadoop fs -put /climate/input    
#文件夹使用-mkdir创建
```

4.运行程序：

命令：

```
hadoop jar MyMaxMin.jar MyMaxMin /climate/input/2015-TX_Austin /climete/output
#出现successfully则表示可以了
```

5.查看或者下载结果：输出**cold day**

```
Hadoop fs -cat/-get /climate/output/*
```

![image-20221110205750269](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221110205750269.png)

## 5.输出一年中最低的温度和日期：

​	java文件：

```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

import java.io.IOException;

public class MintemputerGet {
    public static class MinWeatherMapper extends Mapper<LongWritable, Text, Text, DoubleWritable> {
        @Override
        protected void map(LongWritable k1, Text v1, Context context) throws IOException, InterruptedException {
            String line = v1.toString();
//            Double max = null;
            Double min = null;
            try {
                // 获取一行中的气温MAX值
//                max = Double.parseDouble(line.substring(39, 45));
                // 获取一行中的气温MIN值
                min = Double.parseDouble(line.substring(47, 53));
            } catch (NumberFormatException e) {
                // 如果出现异常，则当前的这一个map task不执行，直接返回
                return;
            }
            // 写到context中
//            context.write(new Text("MAX"), new DoubleWritable(max));
            context.write(new Text("全年最低气温"), new DoubleWritable(min));
        }
    }

    public static class MinWeatherReducer extends Reducer<Text, DoubleWritable, Text, DoubleWritable> {
        @Override
        protected void reduce(Text k2, Iterable<DoubleWritable> v2s, Context context) throws IOException, InterruptedException {
            // 先预定义最大和最小气温值
            double max = Double.MIN_VALUE;
            double min = Double.MAX_VALUE;
            // 得到迭代列表中的气温最大值和最小值
            if ("MAX".equals(k2.toString())) {
                for (DoubleWritable v2 : v2s) {
double tmp = v2.get();
                    if (tmp > max) {
                        max = tmp;
                    }
                }
            } else {
                for (DoubleWritable v2 : v2s) {
                    double tmp = v2.get();
                    if (tmp < min) {
                        min = tmp;
                    }
                }
            }
            // 将结果写入到context中
            context.write(k2, "MAX".equals(k2.toString()) ? new DoubleWritable(max) : new DoubleWritable(min));
        }
    }
    public static void main(String[] args) throws Exception {
        if (args == null || args.length < 2) {
            System.err.println("Parameter Errors! Usages:<inputpath> <outputpath>");
            System.exit(-1);
        }

        Path inputPath = new Path(args[0]);
        Path outputPath = new Path(args[1]);

        Configuration conf = new Configuration();
        String jobName = MintemputerGet.class.getSimpleName();
        Job job = Job.getInstance(conf, jobName);
        //设置job运行的jar
        job.setJarByClass(MintemputerGet.class);
        //设置整个程序的输入
        FileInputFormat.setInputPaths(job,inputPath);
        job.setInputFormatClass(TextInputFormat.class);//就是设置如何将输入文件解析成一行一行内容的解析类
        //设置mapper
        job.setMapperClass(MinWeatherMapper.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(DoubleWritable.class);
        //设置整个程序的输出
        // outputpath.getFileSystem(conf).delete(outputpath, true);//如果当前输出目录存在，删除之，以避免.FileAlreadyExistsException
        FileOutputFormat.setOutputPath(job,outputPath );
        job.setOutputFormatClass(TextOutputFormat.class);
        //设置reducer
        job.setReducerClass(MinWeatherReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(DoubleWritable.class);


        //指定程序有几个reducer去运行
        job.setNumReduceTasks(1);
        //提交程序
        job.waitForCompletion(true);
    }
}

```

结果展示：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221110211904929.png" alt="image-20221110211904929" style="zoom:150%;" />

其余步骤和上面一样。