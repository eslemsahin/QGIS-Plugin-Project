## FAVORITE LOCATION QGIS PLUGIN

--------

## The Project Implementation Video

https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/7d9b6412-9707-4871-97a4-5b6730bd78b0

--------

## PLUGIN CREATION SECTION

In line with the given task, I developed a QGIS plugin for my favorite city. Below, you can find the details of this process in sequence.

- Firstly, the creation of the plugin was initiated through the "Plugin Builder" option under the "Plugins" menu in the QGIS application.

- The necessary fields in the form, such as the class name, plugin name, and a brief description, were filled out as shown in the visual.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/564304c9-4cfa-4bcb-b277-892dd3fe4bab)

- The required description for the About dialog box of the plugin was added.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/1e7d4810-8148-4534-b04d-498032d83dec)

- After selecting the directory where the plugin files would be created, a folder was specified and the plugin was generated.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/4891534a-c62d-48fd-af68-13c49174eced)

- With these steps, the plugin was prepared using the QGIS Plugin Builder tool.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/3a5d638f-23c1-42ff-8598-77b2aa7aff02)

Afterward, to be able to use the developed plugin, it was necessary to copy the plugin directory to the folder where QGIS stores plugins.

- To locate the current profile folder in QGIS, we followed the path Settings > User Profiles > Open Active Profile Folder. Once the profile folder was found, the plugin directory was copied to the python > plugins subfolder.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/2631d394-90a2-4c4f-b262-27f76f5ae402)

- Following the copy operation, the newly created plugin was enabled in the Installed tab of QGIS Plugins > Manage and Install Plugins.
  
  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/c2ea46f6-49fc-4b6d-91af-913b913acd1f)

- Later, to customize the window that appears when the plugin is invoked, the ui extension file for the plugin was added from Qt Designer, and a Eslem Button with a push bottom and text was placed.

  ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/f20c25e0-9a57-421f-8793-f92f1b0fb028)

 - In addition, all new updates made in the created plugin file are integrated into the plugin with the Plugin Reloader plugin in QGIS.

   ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/7a4f30b9-930e-4ebe-b5a5-f0400e7658a3)

Thus, the process of creating a QGIS Plugin as Favorite Location has been completed.

---------

## DEVELOP A QGIS PLUGIN SECTION


The first step in using the prepared plugin according to its functionality was to obtain the necessary data.

- Since reaching the favorite location involves utilizing data from two layers:

  * Firstly, the vector data containing the boundaries of Turkey was obtained from the Harita Genel Müdürlüğü website. (https://www.harita.gov.tr/urun/turkiye-mulki-idare-sinirlari/232)

  * Following that, the spatial data for the favorite city in Turkey, specifically "Yozgat province," was acquired as a shapefile from the QGIS OSM Place Search plugin.
 
  * ![image](https://github.com/GMT-456-GIS-Programming/hello-qgis-plugins-eslemsahin/assets/120361919/8764d545-957f-4eae-8724-4447e02fb527)


Subsequently second step, the coding part in the "favorite_location_dialog.py" file was completed to ensure the proper functionality of the plugin.

Below you can follow the codes and explanations required for the plugin to work.

- Imports the necessary libraries. os is used to handle file paths, uic is used to load .ui files created by Qt Designer, QtWidgets is used for PyQt interface elements, and QgsProject, QgsVectorLayer is used for QGIS project and vector layers.
  
```
import os
from qgis.PyQt import uic
from qgis.PyQt import QtWidgets
from qgis.core import QgsProject, QgsVectorLayer
```

- Loads the .ui file created with Qt Designer and assigns the generated class to the variable FORM_CLASS.

```
FORM_CLASS, _ = uic.loadUiType(os.path.join(
    os.path.dirname(__file__), 'favorite_location_dialog_base.ui'))
```

 - Defines a class named FavoriteLocationDialog that inherits from QtWidgets.QDialog and FORM_CLASS.

```
class FavoriteLocationDialog(QtWidgets.QDialog, FORM_CLASS):
    def __init__(self, parent=None):
```

 - The constructor function manages the initialization of the class. The setupUi(self) function loads the interface elements created by Qt Designer. It then connects the click event of the pushButton to the on_pushButton_clicked function. The layers_added flag tracks whether layers have been added. The shapefiles list contains information about the shapefiles to be added.

```
        super(FavoriteLocationDialog, self).__init__(parent)
        self.setupUi(self)
        self.pushButton.clicked.connect(self.on_pushButton_clicked)
        self.layers_added = False
        self.shapefiles = [
            {'path': 'C:/Users/ASUS-PC/Desktop/TR/TR.shp', 'name': 'TR'},
            {'path': 'C:/Users/ASUS-PC/Desktop/Yozgat/Yozgat.shp', 'name': 'Yozgat'}
        ]

```

 - This function creates a vector layer with the given file path and layer name and adds it to the QGIS project.

```
    def add_vector_layer(self, path, name):
        layer = QgsVectorLayer(path, name, 'ogr')
        if not layer.isValid():
            print(f'Invalid layer "{name}". Check file path or layer format.')
            return False
        QgsProject.instance().addMapLayer(layer)
        return True
```

- This function manages the events that occur when the pushButton is clicked. It checks the layers_added flag, and if layers have not been added yet, it adds the shapefiles from the shapefiles list and sets the layers_added flag to True. If the layers are already added, it informs the user.
  
```
    def on_pushButton_clicked(self):
        if not self.layers_added:
            layers_added_successfully = True
            for shapefile in self.shapefiles:
                path, name = shapefile['path'], shapefile['name']
                if not self.add_vector_layer(path, name):
                    layers_added_successfully = False

            if layers_added_successfully:
                print('Layers added successfully.')
                self.layers_added = True
            else:
                print('Some layers could not be added. Please check and try again.')

        else:
            print('Layers are already added. No further action is taken.')

```

After the changes made with this coding are reloaded in QGIS with Plugin Reloader, the plugin is run and the favorite location is obtained, as shown in the video in the " The Project Implementation Video " section. 


