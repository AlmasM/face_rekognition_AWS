# Project Name: face_rekognition_AWS
Setting up and calling AWS Face Rekognition API through Raspberry PI


### Outline
1. [Introduction](#introduction)
2. [Set up AWS on Raspberry PI](#set-up-aws-on-raspberry-pi)
3. [Camera Set Up on Raspberry PI](#camera-set-up-on-raspberry-pi)
4. [Upload Images to Amazon S3 Bucket](#upload-images-to-s3-bucket)
5. [Upload Faces to Collection](#upload-faces-to-collection)
6. [Identify Faces from Collection, Determine Emotions and Display on Web browser](identify-faces-from-collection-determin-emotions-and-display-on-web-browser)
   - Identifying Emotions
   - Upload images to S3 bucket and create collection
   - Finding faces in the Database
   - Opening Web Browser using Python
5. [Putting it together](putting-it-all-together)

### Introduction
Following repo will provide a guide on how to set up your Amazon AWS Rekognition, set up camera on Raspberry PI, 
identify emotions of the people in the picture and classify each face from known database.

### Set up AWS on Raspberry PI

Installation is straighforward and pretty easy when you follow [Installation Guide]( http://boto3.readthedocs.io/en/latest/guide/quickstart.html#installation). Then, set up [configuration details]( http://boto3.readthedocs.io/en/latest/guide/quickstart.html#configuration)
 

### Camera Set Up on Raspberry PI

  - **Hardware**
      
      Installing and attaching camera on Raspberry PI is straightforward, which can be found [here](https://projects.raspberrypi.org/en/projects/getting-started-with-picamera/4)
  - **Software**
  
    Python code `camera.py` will start camera on Raspberry PI through web browser. This python code already includes HTML code and some CSS
    codes
    
    **Note** You should have *PIL* (Python Image Library), *boto3*, *picamera* packages installed
    
    In order to run camera.py type in command prompt
    ```
    python3 camera.py
    ```
    Then, open preferred web browser and type 
    ```
    http://0.0.0.0:8000/index.html
    ```
    
    Once you snap the picture on your web browser, it will create and save image **currIm.jpg**
    
 ### Upload Images to S3 Bucket
 1. Create bucket on AWS S3. Then open `upload_s3.py` file in text editor
 
 2. Go to your Console on AWS and find AWS Access and Secret keys. Then, add Access and Secret keys to `upload_s3.py` file
    ```
    AWS_ACCESS_KEY = Your Access Key
    AWS_ACCESS_SECRET_KEY = Your Secret Key 
    ```
3. From file `upload_s3.py` you can call function `upload_main(fileName, bucket)` by providing name of the file you want uploaded


### Upload Faces to Collection

**Note** I found concept of `collection` a bit confusing at first. Essentially, every `bucket` can contain a `collection`. Moving forward, you will create bucket in S3 and upload your files into it. Bucket will contain physical copies of your files. Then, using those files in S3 bucket, you will create `collection` items, which can be characterized as virtually grouped items in S3

Open file `collection_faces.py` and do the following:

 - Replace variable value `rootdir` to a *directory* where all the faces are located. In this case, all the people you want to have in your
 dataset as *known people*
 - Replace `bucket` and `collectionId` with bucket and collection where you want to store your dataset of known people.
 - Helper functions you can (un)comment:
   - `list_collection()` will list all the collections in your Amazon AWS
   - `list_items_collection(collectionId, bucket)` takes 2 arguments (collection name and bucket name), and will return list of items
   in the collection & bucket specified.
 - Function `get_bucket_names(bucket)` takes 1 argument (bucket name) and returns a *list* of all the items in the bucket. This function will
 be used in the next step.
 - Function `add_faces_collection(collectionId, photo_l, bucket)` will take items from `photo_l` list and put them in collection
 and bucket you specified. **Result** will be a bucket & collection which will contain your *known people* dataset

**Note** The following code has been used from [Amazon AWS Rekognition Documentation](https://docs.aws.amazon.com/rekognition/latest/dg/create-collection-procedure.html)

### Identify Faces from Collection, Determine Emotions and Display on Web browser

**Note**: make sure to install opencv `pip3 install opencv-python`

When you run the following code in your terminal `python test_collection.py`, the code will do the following:
  1. It will wait until `currIm.jpg` file appears in the folder
  2. Then it will run function `recognition_main()` which identifies all the new faces, and compares each new face with faces in your collection
     - In other words, suppose you created collection `myCollection` which contains some # of faces. Suppose file `currIm.jpg` already exists.
     After `recognition_main()` function is called all the faces identified in the `currIm.jpg` will be added to `myCollection` and compared
     with **all the faces in the collection**. 
     - In Amazon AWS (as far as I know) in order to recognize and identify many faces in the image, you have to upload all the images to 
     the same collection. Example: Suppose your collection `myCollection` contains 100 people. Also, suppose new picture you uploaded 
     is a group picture that has 5 faces, and you want to see which of these 5 people are in `myCollection`. In order to do that, **you have to
     upload all the faces the same collection** 
  3. Once all the faces are identified, it will run `emotion_main()` function, which is located in `demo.py` python file. 
     - In the `demo.py` file, replace `bucket` variable inside `emotion_main()` function
     - The result will be, new file `currIm_emotion.jpg` with all the emotions identified for each face in the `currIm.jpg` image.
  4. Once both `emotion_main()` and `recognition_main()` executed, the result will be projected on web browser of your choosing *automatically*. 
  
 
 **Note** The following code has been used from [Amazon AWS Rekognition Documentation](https://docs.aws.amazon.com/rekognition/latest/dg/search-face-with-id-procedure.html)


### Putting it all together
1. Open terminal and type `python3 camera.py`
2. Open web browser and type `http://0.0.0.0:8000/index.html`
3. Open new terminal window, or new tab and type `python test_collection.py`
   - You can also run both `camera.py` and `test_collection.py` in the same terminal window. In that case, type 
   ```
   $ python camera.py &
   $ python test_collection.py
   ```
 4. **Result** webpage with 3 images: original image (currIm.jpg), identified faces image, and emotion + identified faces
