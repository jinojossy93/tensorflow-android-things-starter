# Overview

This repo contains a simplified and trimmed down version of tensorflow's
[android image classification example](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/android)
in the `android/` directory.

The `scripts` directory contains helpers for the codelab.

The following are steps to complete the tensorflow tutorial using an Ubuntu 16.04 and Android Studio 3.0
I think it will work with others too.. Just lletting you know what have I used to successfully complete this

I did face a lot of issues while using codelab that google provided [here](https://codelabs.developers.google.com/codelabs/tensorflow-for-poets-2)
This project is based on the [TensorFlow Android Camera Demo TF_Classify app](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/android/).
So I made this to make it lot easier..

Hope you guys like it.

#Make sure you have installed tensorflow for python3 using pip3 install
# use virtual envionment(tensorflow-dev)
#Test the model
python3 -m scripts.label_image \
  --graph=tf_files/retrained_graph.pb  \
  --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg
#Optimize the model
#Optimize for inference
python3 -m tensorflow.python.tools.optimize_for_inference \
  --input=tf_files/retrained_graph.pb \
  --output=tf_files/optimized_graph.pb \
  --input_names="input" \
  --output_names="final_result"
#Verify the optimized model
python3 -m scripts.label_image \
  --graph=tf_files/retrained_graph.pb\
  --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg
python3 -m scripts.label_image \
    --graph=tf_files/optimized_graph.pb \
    --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg
#Investigate the changes with TensorBoard
pkill -f tensorboard
tensorboard --logdir tf_files/training_summaries &

#Now add your two graphs as TensorBoard logs:
python3 -m scripts.graph_pb2tb tf_files/training_summaries/retrained \
  tf_files/retrained_graph.pb 

python3 -m scripts.graph_pb2tb tf_files/training_summaries/optimized \
  tf_files/optimized_graph.pb 
#Make the model compressible
#check compressions
gzip -c tf_files/optimized_graph.pb > tf_files/optimized_graph.pb.gz
gzip -l tf_files/optimized_graph.pb.gz
#Quantize the network weights
python3 -m scripts.quantize_graph \
  --input=tf_files/optimized_graph.pb \
  --output=tf_files/rounded_graph.pb \
  --output_node_names=final_result \
  --mode=weights_rounded
#check compressions
gzip -c tf_files/optimized_graph.pb > tf_files/optimized_graph.pb.gz
gzip -l tf_files/optimized_graph.pb.gz
#Compare with the image
python3 -m scripts.label_image   --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg   --graph=tf_files/optimized_graph.pb
python3 -m scripts.label_image \
  --image=tf_files/flower_photos/daisy/3475870145_685a19116d.jpg \
  --graph=tf_files/rounded_graph.pb
#you will have to copy 200 Mb sized full flower_photoz folder
#Now its time to evaluate the performance
python3 -m scripts.evaluate  tf_files/optimized_graph.pb
python3 -m scripts.evaluate  tf_files/rounded_graph.pb
#Add your model files to the project
cp tf_files/rounded_graph.pb android/tfmobile/assets/graph.pb
cp tf_files/retrained_labels.txt android/tfmobile/assets/labels.txt 

# Open the tfmobile folder in Android studio
#change the following in project directory
ClassifierActivity.java

  private static final String INPUT_NAME = "input";
  private static final String OUTPUT_NAME = "final_result";

