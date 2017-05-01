# Final Project at Metis - The Mural Networks

Street art has evoloved from text based graffiti to complex form of artistic expressions. In 80s and 90s, street art could be viewed as a form of Vandalism but today, street artists often collaborate with property owners, clothing brands, galleries, and non-profit organizations to promote products, send messages to the public, or to promote their artworks. The market for street art has been booming for last few years and number of NY's popular street artists are invited for an exhibition overseas. As a street art fan, I wanted to create something related to New York and contribute to the art + technology field, which I think is still somewhat new.

If interested, check out some articles related to street art market booming:
https://www.wsj.com/articles/street-art-is-finding-a-home-with-collectors-1442800929
http://nypost.com/2015/10/10/where-did-all-of-banksys-nyc-art-go/
http://www.koreaherald.com/view.php?ud=20161211000253
https://hypebeast.com/2016/10/jerkface-interview-saturday-morning-exhibit


### Motivation

So imagine you are walking down the street and see beautiful street arts. If you are a street art enthusiast like me, you might wish to learn about the artist. But there isn’t any easy way to do so. 

New York is filled with beautiful and colorful street arts. They could be anything from prints, stickers, mural, graffiti, and even installation art. What’s interesting is that each street artist has very unique style in color, patterns, or materials used. So I thought this could be an interesting computer vision classification problem where I could utilize deep learning neural networks. 

### Data Gathering & Cleaning

I follow number of different street artists on Instagram. Street artists today use social media as a platform to show their past artworks or upcoming projects. So Instagram is the perfect place to get the street art image data!

I first tried Instagram API then realized there are so many restrictions (as expected). It seems like I always try to go for API then end up scraping on my own. Then, I found this cool tool called 'Instagram-Scraper' where I could simply scrape all images from an Instagram public user to my local machine. You can check out the tool <a href ='https://github.com/rarcega/instagram-scraper'> here </a>. Now the next step was cleaning the images. I cropped out other details like people and building so that I only have street art part left. I had about 5000 images to work with as a result. 

### Model

I used Keras with backend Theano for Convolusional Neural Networks. I was new to Keras so I tried cat/dog image classification exercise to practice and I could see how powerful Keras can be. 

```python
from keras.preprocessing.image import ImageDataGenerator, array_to_img, img_to_array, load_img
from keras.models import Sequential, load_model
from keras.layers import Conv2D, MaxPooling2D
from keras.layers import Activation, Dropout, Flatten, Dense
from keras import backend as K

K.set_image_dim_ordering('tf')

from keras.utils import np_utils
import cv2

import os
from natsort import natsorted
import numpy as np
from sklearn.utils import shuffle
from sklearn.model_selection import KFold,train_test_split
from matplotlib import pyplot as plt

# Train and Validation directory structure matters when using ImageDataGenerator!
train_data_dir = 'data/train/'
validation_data_dir = 'data/test/'
epochs = 100
batch_size = 32
```

I will have about 500 images for each artist. It's relatively small number of images so I used image augmentation where ImageDataGenerator randomly zooms in/out and rotate images to create multiple samples from one. 

```python
# IMAGE AUGEMENTATION
# Random rotation, shifts, shear and flips

train_datagen = ImageDataGenerator(
    rotation_range=40,
    width_shift_range=0.2,
    height_shift_range=0.2,
    shear_range=0.2,
    zoom_range=0.5,
    horizontal_flip= True,
    fill_mode = 'nearest',
    rescale=1. / 255)

test_datagen = ImageDataGenerator(
    rescale=1. / 255
)


train_generator = train_datagen.flow_from_directory(
    train_data_dir,
    target_size = (img_width,img_height),
    batch_size=batch_size,
    class_mode = 'categorical'
    # class_mode = 'binary' for 2 classes
)

validation_generator = test_datagen.flow_from_directory(
    validation_data_dir,
    target_size = (img_width,img_height),
    batch_size=batch_size,
    class_mode = 'categorical'
)
```

Here's what my CNN looks like,

```python
model = Sequential()
model.add(Conv2D(32, 3,3, input_shape=input_shape))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2,2)))

model.add(Conv2D(32, 3, 3))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))

model.add(Conv2D(64, 3, 3))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))

model.add(Flatten())
model.add(Dense(64))
model.add(Activation('relu'))
model.add(Dropout(0.5))
model.add(Dense(3))
model.add(Activation('softmax'))
```

It's a three convolusional layers with relu followed by a softmax. 

![alt text](/images/cnnsvg.png "cnn_svg")

Now training the model! Using GPU on AWS made it easier for me to run it multiple times and modify parameters. 

```python
model.compile(loss='categorical_crossentropy', optimizer='rmsprop', metrics=['accuracy'])
model.fit_generator(
    train_generator,
    samples_per_epoch= 288,
    nb_epoch=epochs,
    validation_data = validation_generator,
    nb_val_samples=110,
    verbose=1
)
```

Two artists, 
```
Epoch 45/45
846/846 [==============================] - 424s - loss: 0.3004 - acc: 0.8700 - val_loss: 0.1236 - val_acc: 0.9841
```

Three artists, 
```
Epoch 100/100
288/288 [==============================] - 256s - loss: 0.6802 - acc: 0.7292 - val_loss: 0.4845 - val_acc: 0.7818
```

Four artists, 
```
Epoch 45/45
846/846 [==============================] - 352s - loss: 0.7404 - acc: 0.7001 - val_loss: 0.5975 - val_acc: 0.7529
```

My binary classificaion had a really good accuracy but adding more data from more artists decreased the accuracy gradually. I could try increasing epoch but if I were to have 10 different artists, this would need serious fine-tuning. To do so, leveraging a pre-trained model would make more sense. For my next post about this project, I will post about utilizing VGG16 archetecture and boosting the accuracy on more artists. 

### The App

My final product would be a web app with above model running in the back-end. As soon as I finish fine-tuning the model, I will have it on the web for you to play around with. For now, it only runs on my local and here is how it looks like,

![alt text](/images/muralnetworks.png "mural_networks")


This is the code for the front-end.

```html
<!doctype html>
<html>
<head>
	<meta charset=utf-8>
	<title>The Mural Networks</title>
	<h5><font color = "#eaece5">The Mural Networks v0.0.1<font></h5>
	<style>
		html {
		  	height: 100%;
		}
		body {
			height: 100%;
		    background-color: #282828;
		    background-image:linear-gradient(#434343, #282828);
		}
		#content{
		    background-color: transparent;
		    background-image: linear-gradient(0deg, transparent 24%, rgba(255, 255, 255, .05) 25%, rgba(255, 255, 255, .05) 26%, transparent 27%, transparent 74%, rgba(255, 255, 255, .05) 75%, rgba(255, 255, 255, .05) 76%, transparent 77%, transparent), linear-gradient(90deg, transparent 24%, rgba(255, 255, 255, .05) 25%, rgba(255, 255, 255, .05) 26%, transparent 27%, transparent 74%, rgba(255, 255, 255, .05) 75%, rgba(255, 255, 255, .05) 76%, transparent 77%, transparent);
			height:100%;
		  	background-size:50px 50px;
		}
  		#img_wrapper1{

  			border: 3px solid #eaece5; 
  			width: 600px; 
  			height: 400px;
  			background-color: white;
  		}
  		#img_wrapper2{

  			border: 3px solid #eaece5; 
  			width: 300px; 
  			height: 300px;
  			background-color: white;

  		}
}
	</style>

	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
	<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/croppie/2.4.1/croppie.css" />
	<script src="https://cdnjs.cloudflare.com/ajax/libs/croppie/2.4.1/croppie.js"></script>
</head>
<body>
<div id="content" style="height:100%" align="center">
	<form method=post enctype=multipart/form-data>
	<h1><font color = "#eaece5">Upload Your Street Art Image</font></h1>

	<p>
		<input id=browse_button type=file name=file style="color: white" >
		<input id=submit_button type=submit value=Predict >
		<div> <font color ='#eaece5'>Zoom-in and crop your image for more accurate result!</font></div>
	</p>
</form>

<div id = 'img_wrapper1'>
	<img id='myimg' style="display: block;margin:auto; max-width: 600px; max-height:400px; width:auto; height:auto; position:absolute; margin-left:auto; margin-right:auto; left:0;right:0;" alt="" />
</div>
<div id = 'img_wrapper2' style='display: none'>
	<img id='cropped_img'>
</div>
<script type="text/javascript">
	var el = document.getElementById('myimg');

	var basic = new Croppie(el, {
	    viewport: { width: 300, height: 300, type: 'square' },
	    boundary: { width: 600, height: 400 },
	    showZoomer: true,
	    enableOrientation: true
	});

	function readURL(input) {

	    if (input.files && input.files[0]) {
	        var reader = new FileReader();
	        reader.onload = function (e) {

	            //$('#myimg').attr('src', e.target.result);
        		basic.bind(e.target.result).then(function(src) {

        			console.log(file.name)
        			$('#cropped_img').attr('src', file.name);
				    
				});
	        }

	        reader.readAsDataURL(input.files[0]);
	    }
	}

   $("#browse_button").change(function(){
        readURL(this);
        basic.desctroy;
    })

	function draw() {
		
		document.getElementById('myimg').src = "static/" + document.getElementById('browse_button').value;
	}

	$(document).ready(function(){

		$("#submit_button").click(function(){
			console.log("success!")
			
			basic.result({type: 'canvas', format: 'jpeg'}).then(function(src) {
				
				// SAVE CROPPED IMAGE HERE!
				$('#result_img').attr('src', src);
				$('#img_input').attr('src',src);
				console.log(src)
				//draw();
			});
		});
	});

</script>

</div>
</body>
</html>
```

Now the backend looks like below. THe model is loaded in the beginning and the predict button would trigger the action and make prediction. Please note that I deleted the code for providing artists information here just because I do not want to disclose which street artists I am testing with.  

```python
from keras.preprocessing.image import ImageDataGenerator, array_to_img, img_to_array, load_img
from keras.models import Sequential, load_model
from keras.layers import Conv2D, MaxPooling2D
from keras.layers import Activation, Dropout, Flatten, Dense
from keras import backend as K
import cv2
import numpy as np
import base64


import os
from flask import Flask, request, redirect, url_for, send_from_directory
from werkzeug.utils import secure_filename


model = load_model('../artists_100epoch.h5')
model.compile(loss='categorical_crossentropy', optimizer='rmsprop', metrics=['accuracy'])

img_path = '/Users/myusername/Metis/final_project/static'


ALLOWED_EXTENSIONS = set(['jpg', 'jpeg'])


app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = img_path
# Homepage

@app.route('/')
def viz_page():
    with open("index.html", 'r') as viz_file:
		return viz_file.read()

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET','POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            return 'No file part'
        file = request.files['file']

        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            print 'file selected: ', filename
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))

            # Take img input and make prediction here
            # ---------------------------------------

            if make_prediction(img_path, filename) == 1:
            # Provide artists information code here
            # Deleted this part on purpose


# Using CV2 to resize the image and make prediction
def make_prediction(img_path, filename):
    predict_img = cv2.resize(cv2.imread(img_path + '/' + filename), (300, 300)).astype(np.float32)
    predict_img = np.expand_dims(predict_img, axis=0)
    classes = model.predict_classes(predict_img, batch_size= 1, verbose=1)

    return classes

if __name__ == "__main__":

	app.run('0.0.0.0')
```

This app works nicely! I added zoom-in plugins so the model could produce more accurate data. 

### Future Works

First thing would be boosting accuracy by fine-tuning the model. I do not want to misinform users by providing false information here. Therefore, the model should be very accurate and be trained on a large, good quality image dataset. If this part works, the app can be converted into a mobile-friendly version where I could start testing on my mobile devices. 

Once they are done, I will post it on my blog so anybody can play around with.

<a href='https://github.com/jjchoi08/DSProj/blob/master/proj_kojac.pdf'>Here </a> is a link for my final presentation in the mean time. 

Stay Tune!
