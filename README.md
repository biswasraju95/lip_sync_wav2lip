# lip_sync_wav2lip
Wav2Lip Sync is an innovative open-source project that harnesses the power of the state-of-the-art Wav2Lip algorithm to achieve real-time lip synchronization with unprecedented accuracy. The project aims to revolutionize lip-syncing capabilities for various applications, including video editing, dubbing, virtual characters, and more. 
Lip syncing has always been a challenging task in the realm of multimedia, often requiring significant manual effort and precise adjustments. However, with Wav2Lip Sync, this process becomes effortless and efficient. The algorithm takes advantage of Deep Learning techniques to automatically align lip movements in a target video to match a given audio source.

##Prerequisites :


Python 3.6
ffmpeg: sudo apt-get install ffmpeg
Install necessary packages using pip install -r requirements.txt.
Face detection pre-trained model should be downloaded to face_detection/detection/sfd/s3fd.pth. Alternative link if the above does not work.

##Tips for better results:


Experiment with the --pads argument to adjust the detected face bounding box. Often leads to improved results. You might need to increase the bottom padding to include the chin region. E.g. --pads 0 20 0 0.
If you see the mouth position dislocated or some weird artifacts such as two mouths, then it can be because of over-smoothing the face detections. Use the --nosmooth argument and give it another try.
Experiment with the --resize_factor argument, to get a lower resolutionlower-resolution video. Why? The models are trained on faces whichthat were at a lower resolution. You might get better, visually pleasing results for 720p videos than for 1080p videos (in many cases, the latter works well too).
The Wav2Lip model without GAN usually needs more experimenting with the above two to get the most ideal results, and sometimes, can give you a better result as well.

##Preparing LRS2 for training


Our models are trained on LRS2
data_root (mvlrs_v1)
├── main, pretrain (we use only main folder in this work)
|	├── list of folders
|	│   ├── five-digit numbered video IDs ending with (.mp4)

Place the LRS2 filelists (train, val, test) .txt files in the filelists/ folder.


##Preprocess the dataset for fast training


python preprocess.py --data_root data_root/main --preprocessed_root lrs2_preprocessed/
Additional options like batch_size and the number of GPUs to use in parallel to use can also be set.


##Preprocessed LRS2 folder structure


preprocessed_root (lrs2_preprocessed)
├── list of folders
|	├── Folders with five-digit numbered video IDs
|	│   ├── *.jpg
|	│   ├── audio.wav



##Train!


There are two major steps: (i) Train the expert lip-sync discriminator, (ii) Train the Wav2Lip model(s).



##Training the expert discriminator


You can download the pre-trained weights if you want to skip this step. To train it:

python color_syncnet_train.py --data_root lrs2_preprocessed/ --checkpoint_dir <folder_to_save_checkpoints>



##Training the Wav2Lip models



You can either train the model without the additional visual quality disriminatordiscriminator (< 1 day of training) or use the discriminator (~2 days). For the former, run:

python wav2lip_train.py --data_root lrs2_preprocessed/ --checkpoint_dir <folder_to_save_checkpoints> --syncnet_checkpoint_path <path_to_expert_disc_checkpoint>
To train with the visual quality discriminator, you should run hq_wav2lip_train.py instead. The arguments for both the files are similar. In both the cases, you can resume training as well. Look at python wav2lip_train.py --help for more details. You can also set additional less commonly-used hyper-parameters at the bottom of the hparams.py file.

Training on datasets other than LRS2
Training on other datasets might require modifications to the code. Please read the following before you raise an issue:

You might not get good results by training/fine-tuning on a few minutes of a single speaker. This is a separate research problem, to which we do not have a solution yet. Thus, we would most likely not be able to resolve your issue.
You must train the expert discriminator for your own dataset before training Wav2Lip.
If it is your own dataset downloaded from the web, in most cases, needs to be sync-corrected.
Be mindful of the FPS of the videos of your dataset. Changes to FPS would need significant code changes.
The expert discriminator's eval loss should go down to ~0.25 and the Wav2Lip eval sync loss should go down to ~0.2 to get good results.
When raising an issue on this topic, please let us know that you are aware of all these points.

We have an HD model trained on a dataset allowing commercial usage. The size of the generated face will be 1280 x 640 in our new model.

