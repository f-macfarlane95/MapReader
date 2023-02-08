Annotate
========

.. contents:: ``MapReader`` provides a flexible way to set up a new annotation task in three simple steps:
	:local:

1. Edit the annotation tasks file.
-------------------------------------------
	
	The ``annotation_tasks.yaml`` file is used to set up your annotation tasks. It contains two sections - ``tasks`` and ``paths``.
	
	The ``tasks`` section is used to specify tasks and their labels. e.g. : 
	   
	.. code :: yaml

				# ---------------------------------------
				# Define an annotation task
				# This includes:
				# 1. a name (e.g. building_simple or rail_space)
				# 2. a list of labels to be used for this task
				# ---------------------------------------
				
				tasks:
				  building_simple:
					labels: ["No", "building"]
				  rail_space:
					labels: ["No", "rail space"]

	The ``paths`` section is used to specify file paths to sets of images you would like to annotate. e.g. :

	.. code :: yaml

		# ---------------------------------------
		# paths
		# You need to specify:
		# 1. a name for the set of images to annotate (e.g. test_one_inch_maps_001)
		# 2. patch_paths: path to all the patches (children) to be annotated
		# 3. parent_paths: path to the parent images
		# 4. annot_dir: directory in which the outputs will be stored
		# ---------------------------------------
		
		paths:
		  test_one_inch_maps_001:
			patch_paths: "./maps/slice_50_50/patch-*PNG"
			parent_paths: "./maps/*png"
			annot_dir: "./annotations_one_inch"
		
2.	Annotate your images.
------------------------------

	To prepare your annotations, you must specify a ``userID``, ``annotation_tasks_file`` (i.e. the ``./annotation_task.yaml`` made in step 1), tell ``MapReader`` which ``task`` you'd like to run and which  ``annotation set`` you would like to run on. 

	To do this, use: 

	.. code :: python

		from mapreader.annotate.utils import prepare_annotation
			
		userID="your_name"
		annotation_tasks_file="./annotation_task.yaml"
		task="rail_space"
		annotation_set="test_one_inch_maps_001"

		annotation=prepare_annotation(userID, annotation_tasks_file, task, annotation_set=annotation_set)

	You can then interactively annotate a sample of your images using:

	.. code :: python

		annotation

	.. image:: ../figures/annotate.png
		:width: 400px

	To help with annotating, you can set the annotation interface to show a context image using ``context_image = True``. This creates a second panel in the annotation interface, showing your patch in the context of a larger region whose size, in pixels, is set by ``xoffset`` and ``yoffset``.
		
	.. code :: python
		
		annotation=prepare_annotation(userID, annotation_tasks_file, task, annotation_set=annotation_set,
					 					context_image=True, xoffset=100, yoffset=100)

		annotation 

	.. image:: ../figures/annotate_context.png
		:width: 400px

	By default, your patches will be shown to you in a random order, but, to help with annotating, can be sorted by their mean pixel intesities using ``sorby = "mean"``.

3.	Save your annotations.
------------------------------
	
	Once you have annotated your sample, you should then save your annotations using:

	.. code :: python

		save_annotation(annotation, userID, task, annotation_tasks_file, annotation_set)

	This saves your annotations as a ``.csv`` file in the ``annot_dir`` specified in your annotation tasks file.