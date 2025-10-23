# Region-Graph-based-Identifier-of-Metaplasia-ReGIoM-
Repository for python tools applied to get a Region Graph-based Identifier of Metaplasia (ReGIoM) that unsupervisely identifies intestinal metaplasia in stomach histology samples. The main script uses diferenrent functions including ```HRgraph.py``` and ```LRgraph.py``` to built High Resolution and Low resolution graphs as presented in ```GastricV3.ipynb```

### This is how it looks a complete execution of the whole pipeline
As shown in figure bellow, the ReGIoM is defined as an unsupervised approach that idntifies intestinal metaplasia by using multiple versions of WholeSlide Images (WSI) at different magnification.This strategy split each WSI into non-squared regions at a low magnification (1×), by using SLIC superpixel estimation, based on intensity valueas of the image at such magnification. Then, each of these regions is subdivided in multiple elements that are finally characterized in terms of shape and texture features, allowing the selection of particular elements as nodes of a high-resolution graph that provides a complete characterization of the tissue given the spatial and features connection between them.

![Model](https://github.com/JonathanTarquino/Region-Graph-based-Identifier-of-Metaplasia-ReGIoM-/blob/main/RegiOM_pipeline.png)
```
i = -1
all_features = []
all_features = pd.DataFrame(all_features)
for cas in cases:
  i = i+1
  if i>1:
    print('Processing case...........', cas[0:-4])
    image_path = PATH_OF_DATA+cas
    patch_size = (2000,2000)
    patches_path = '/content/drive/MyDrive/Gastric_project/URK_data/patches'
    image = openslide.OpenSlide(image_path)

    mag = 1
    # ima, mime_type = image.getThumbnail(width=640, height=480, encoding='PNG')
    # open('thumb.png', 'wb').write(ima)
    # plt.imshow(cv2.imread('thumb.png'))
    # plt.show()
    # image is now a numpy array
    image_name = image_path.split("/")[-1].split(".svs")[0]
    print(patches_path,image_name)

    path = image_path.split(".svs")[0]
    if (os.path.exists(patches_path+image_name+"/") == False):
      os.mkdir(patches_path+image_name+"/")
      print('ok created')
    # print('>>>>>>>>>',image.getMetadata()['sizeX'], image.getMetadata()['sizeY'], path)
    # print(patch_size)
    #get_patches(image, image_name, path, patches_path+image_name+"/", image_path,1)
    unoximage, regions, level = get_image_segment(image, image_name, path, patches_path+image_name+"/", image_path,mag)
    plt.imshow(unoximage)
    plt.show()

    print('Extracting low magnification graph')
    n_superpx = 900
    case_features = LMGraph(unoximage,n_superpx,0,cas,image_path,regions,level)
    case_id_vec = np.zeros((np.shape(case_features)[0],1))
    case_id_vec = np.where(case_id_vec==0,cas,0)
    print('___________________ FEATURES __________________')
    print(case_features,case_id_vec)
    all_features = pd.concat([all_features,case_features])
    print('-----------------------------------------------')
    print(all_features)
```
    all_features.to_csv(r'/content/drive/MyDrive/Gastric_project/URK_data/all_features1.csv', header=None)

