Spot text
=========

MapReader now contains two new methods of spotting text on maps:

- ``DPTextDETRRunner`` - This is used to detect text on maps using `DPTextDETR <https://github.com/ymy-k/DPText-DETR/tree/main>`__ and outputs bounding boxes and scores.
- ``DeepSoloRunner`` - This is used to detect and recognise text on maps using `DeepSolo <https://github.com/ViTAE-Transformer/DeepSolo/tree/main>`__ and outputs bounding boxes, text and scores.

Install dependencies
--------------------

To run these, you will need to install the required dependencies.

.. note::

    We have our own forks of these repos to enable them to work on CPU!

Detectron2
~~~~~~~~~~~

Detectron2 is a popular object detection library built by Facebook AI Research.
The main repo is available `here <https://github.com/facebookresearch/detectron2>`__.

To install, run the following commands in your terminal:

.. code:: bash

    git clone https://github.com/facebookresearch/detectron2.git
    cd detectron2
    pip install .


.. note::

    Since both the DPText-DETR and DeepSolo repos are built ontop of `AdelaiDet <https://github.com/aim-uofa/AdelaiDet>`__, you won't be able to install both at the same. To get around this, you can set up two different conda environments, one for each.

.. admonition:: Detectron2 issues for windows users
    :class: dropdown

    If you are on a windows machine and are having trouble installing Detectron2, you can try the following:
    - Install `Visual Studio Build Tools <https://visualstudio.microsoft.com/downloads/?q=build+tools>`__.
    - Follow instructions `here <https://stackoverflow.com/questions/64261546/how-to-solve-error-microsoft-visual-c-14-0-or-greater-is-required-when-inst>`__ to install the required packages. (The format might be different in newer versions of Visual Studio Build Tools, so you might need to look up the specific package names.)

    Once this is done, retry installing Detectron2.

You should then pick one of the following to install:


DPTextDETR
~~~~~~~~~~~

Our fork for DPText-DETR is available `here <https://github.com/rwood-97/DPText-DETR>`__.

To install, run the following commands in your terminal:

.. code:: bash

    git clone https://github.com/rwood-97/DPText-DETR.git
    cd DPText-DETR
    pip install .

DeepSolo
~~~~~~~~

Our fork for DeepSolo is available `here <https://github.com/rwood-97/DeepSolo>`__

To install, run the following commands in your terminal:

.. code:: bash

    git clone https://github.com/rwood-97/DeepSolo.git
    cd DeepSolo
    pip install .


Advice for patch size
---------------------

When running the text spotting models, we recommend using a patch size of 1024x1024 pixels.
This is the size used in the training of the models, and so should give the best results.

You may also want to create some overlap between your patches as this should minimise cut off text at the edges of patches.
You will need to experiment with the amount of overlap to find the best results for your maps.

.. note::
    Greater overlaps will create more patches and result in greater computational costs when running.

See the :doc:`Load </User-guide/Load>` user guide for more information on how to create patches.

Set-up the runner
-----------------

Once you have installed the dependencies, you can set up your chosen "runner".

You will need to choose a model configuration and download the corresponding model weights.

- Config files can be found in the model repositories under the ``configs`` directory.
- Weights files should be downloaded from the github repositories (links to the downloads are in the README).

e.g. for the ``DPTextDETRRunner``, if you choose the "ArT/R_50_poly.yaml", you should download the "art_final.pth" model weights file from the DPTextDETR repo.

e.g. for the ``DeepSoloRunner``, if you choose the "R_50/IC15/finetune_150k_tt_mlt_13_15_textocr.yaml", you should download the "ic15_res50_finetune_synth-tt-mlt-13-15-textocr.pth" model weights file from the DeepSolo repo.

You will also need to load your patch and parent dataframes.
Assuming you have saved them, as shown in the :doc:`Load </User-guide/Load>` user guide, you can load them like so:

.. code-block:: python

    import pandas as pd

    patch_df = pd.read_csv("patch_df.csv")
    parent_df = pd.read_csv("parent_df.csv")

You can then instantiate your runner.

For the DPTextDETRRunner, use:

.. code-block:: python

    from map_reader.spot_text import DPTextDETRRunner

    my_runner = DeepSoloRunner(
        patch_df,
        parent_df,
        cfg_file = "DPText-DETR/configs/DPText_DETR/ArT/R_50_poly.yaml",
        weights_file = "./art_final.pth",
    )

or, for the DeepSoloRunner, use:

.. code-block:: python

    from map_reader.spot_text import DeepSoloRunner

    my_runner = DeepSoloRunner(
        patch_df,
        parent_df,
        cfg_file = "DeepSolo/configs/R_50/IC15/finetune_150k_tt_mlt_13_15_textocr.yaml",
        weights_file = "./ic15_res50_finetune_synth-tt-mlt-13-15-textocr.pth"
    )

You'll need to adjust the paths to the config and weights files to match your own set-up.

Run the runner
--------------

You can then run the runner on all patches in your patch dataframe:

.. code-block:: python

    patch_preds = my_runner.run_all()

By default, this will return a dictionary containing all the predictions for each patch.
If you'd like to return a dataframe instead, use the ``return_dataframe`` argument:

.. code-block:: python

    patch_preds_df = my_runner.run_all(return_dataframe=True)

MapReader will automatically run a deduplication algorithm to remove overlapping bounding boxes, based on a minimum intersection of area (IoA) for each overlapping polygon.
If two polygons overlap with intersection over area greater than the minimum IoA, the the one with the lower IoA will be kept (i.e. the larger of the two polygons).

Below are two examples of this:

.. image:: ../figures/IoA.png
    :width: 400px

.. image:: ../figures/IoA_0.9.png
    :width: 400px

By default, the minimum IoA is set to 0.7 so the deduplication algorithm will only remove the smaller polygon in the second example.

You can adjust the minimum IoA by setting the ``min_ioa`` argument:

.. code-block:: python

    patch_preds_df = my_runner.run_all(return_dataframe=True, min_ioa=0.9)

Higher ``min_ioa``values will mean a tighter threshold for identifying two polygons as duplicates.

If you'd like to run the runner on a single patch, you can also just run on one image:

.. code-block:: python

    patch_preds = my_runner.run_on_image("path/to/your/image.png", min_ioa=0.7)

Again, this will return a dictionary by default but you can use the ``return_dataframe`` argument to return a dataframe instead.

To view the patch predictions, you can use the ``show`` method.
This takes an image ID as an argument, and will show you all the predictions for that image:

.. code-block:: python

    #EXAMPLE
    my_runner.show(
        "patch-0-0-1000-1000-#map_74488689.png#.png"
    )

By default, this will show the image with the bounding boxes drawn on in red and text in blue.
You can change these by setting the ``border_color`` and ``text_color`` arguments:

.. code-block:: python

    my_runner.show(
        "patch-0-0-1000-1000-#map_74488689.png#.png",
        border_color = "green",
        text_color = "yellow",
    )

You can also change the size of the figure with the ``figsize`` argument.


Scale-up to whole map
---------------------

Once you've got your patch-level predictions, you can scale these up to the parent image using the ``convert_to_parent_pixel_bounds`` method:

.. code-block:: python

    parent_preds = my_runner.convert_to_parent_pixel_bounds()

This will return a dictionary containing the predictions for the parent image.
If you'd like to return a dataframe instead, use the ``return_dataframe`` argument:

.. code-block:: python

    parent_preds_df = my_runner.convert_to_parent_pixel_bounds(return_dataframe=True)

If you have created patches with overlap, then you should deduplicate at the parent level as well.
You can do this by setting the ``deduplicate`` argument and passing a ``min_ioa`` value:

.. code-block:: python

    parent_preds_df = my_runner.convert_to_parent_pixel_bounds(return_dataframe=True, deduplicate=True, min_ioa=0.7)

This will help resolve any issues with predictions being cut-off at the edges of patches since the overlap should help find the full piece of text.

Again, to view the predictions, you can use the ``show`` method.
You should pass a parent image ID as the ``image_id`` argument:

.. code-block:: python

    #EXAMPLE
    my_runner.show(
        "map_74488689.png"
    )

As above, use the ``border_color``, ``text_color`` and ``figsize`` arguments to customize the appearance of the image.

.. code-block:: python

    my_runner.show(
        "map_74488689.png",
        border_color = "green",
        text_color = "yellow",
        figsize = (20, 20),
    )


You can then save these predictions to a csv file:

.. code-block:: python

    parent_preds_df.to_csv("text_preds.csv")

Geo-reference
-------------

If you maps are georeferenced in your ``parent_df``, you can also convert the pixel bounds to georeferenced coordinates using the ``convert_to_coords`` method:

.. code-block:: python

    geo_preds_df = my_runner.convert_to_coords(return_dataframe=True)

Again, you can save these to a csv file as above, or, you can save them to a geojson file for loading into GIS software:

.. code-block:: python

    my_runner.save_to_geojson("text_preds.geojson")

This will save the predictions to a geojson file, with each text prediction as a separate feature.
