Repository for the research paper, "Detecting window-to-wall ratio for urban-scale building simulations using deep learning with street view imagery and an automatic classification algorithm".

Journal: Building Simulation (in press 2025)

Authors: Suppa, A.R., Aliberti, A., Bottero, M.C., & Corrado, V.
<pre>
   
</pre>

The workflow uses the following 8 components:
1. Prepare GIS file:
   - Identifies which building line segments are street-facing where orthogonal images from Google Street View (GSV) are expected.
   - Required inputs: XLS/CSV file exported from GIS where building polygons are split into line segments.
   - Uses a Python-based Jupyter notebook.
2. Extract and stitch GSV images:
   - Automatically downloads and stitches images from GSV, which requires a Google Cloud API Key.
   - Required inputs: XLS/CSV file exported from GIS containing end- and center-point coordinates of street-facing lines, as well as their line length and bearing.
   - Uses a Python-based Jupyter notebook.
3. Annotate images using the custom tool within [Rhino software](https://www.rhino3d.com/):
   - Extracted image files are manually annotated using the custom tool, permitting creation of orthogonal gridlines to locate occluded glazing, and faciltate precise copy/pasting using the 'object snapping' feature of Rhino.
   - The folder contains a developed Rhino template (.3dm) file where annotations are completed, using the different layers in the template for each object (e.g. 2nd-level windows, 2nd-level balcony doors, etc.)
   - The folder also contains a developed Grasshopper binary (.gh) file containing custom Python scripts to analyze annotations and automatically convert them to XML format.
4. Three post-scripts to run NMS and stratify glazing detections:
   - (i) Runs non-maximum suppression between classes for detected window and glazing objects, since YOLOv9 used in the research only runs NMS on each class, thus suppressing a glazing or garage object with the lower confidence score, in the case two such objects overlap.
   - (ii) Runs NMS to exclude glazing and garage detections outside the detected wall area.
   - (iii) Stratifies detections into ground-level and above-ground-level, since the above-ground detections are used to estimate WWR at rear of buildings.
   - Required inputs: JSON files from the deep learning model, assembled and converted to a Pandas DataFrame, with bounding boxes formatted as (xmin, ymin, xmax, ymax).
   - Uses a Python-based Jupyter notebook.
5. Automatic classification algorithm, containing two sub-steps:
   - (i) Classify each building line segment as an additional street-facing fronts or non-street-facing side/rear. All of these segments have no GSV images and thus have no WWR detected from the deep learning model.
   - (ii) Assign a WWR to the above segments. Additional street-facing fronts are assigned the mean WWR value of all street-facing fronts. Rear segments are assigned the WWR of mean detected above-ground-level WWRs. Sides are set with WWR of 5% if they meet minimum lateral distances between buildings, as per Italian urban planning laws, which can be modified to suit local conditions, or 0% if they do not meet minimum legal distances. Sides attached to a neighboring building have zero glazing and are excluded from WWR calculations.
   - Required inputs: XLS/CSV files with (i) the stratified WWR detections and (ii) the original prepared GIS file with the split line segments and identified fronts.
   - Uses a Python-based Jupyter notebook.
7. Estimate building height and number of stories in detected images:
   - In the case study and in other regions, the OSM database is missing data for building height and number of stories.
   - In the present research, OSM buildings were detected for height and stories if the street-facing fronts were within +/- 10% of the corresponding segments in the municipal database.
   - In absence of a municipal database, a method could be employed similar to that described in [Tarkhan et al. (2024)](https://doi.org/10.1016/j.scs.2024.105280) to manually check measurements in Google Earth during image review and cropping stage, thereby ensuring that the line lengths in the OSM data match those present in the images.
   - Required inputs: XLS/CSV file with all building line segments classified and assigned WWRs.
   - Uses a Python-based Jupyter notebook.
8. Convert per-segment to per-orientation WWR detections:
   - UBEM software such as UMI, URBANopt, CitySim, and others require glazing expressed as percentages on the north, east, south, and west orientations.
   - The workflow described above obtained WWR for each line segment of each building, and these are here converted to a WWR-per-orientation basis.
   - Required inputs: XLS/CSV file with all building line segments classified and assigned WWRs, plus estimated height and stories if such data are otherwise not available.
   - Uses a Python-based Jupyter notebook.
9. Urban building energy model (UBEM) within Rhino software:
   - Built with [Dragonfly tools](https://www.ladybug.tools/dragonfly.html) and the [URBANopt module](https://docs.urbanopt.net/) to run individual building simulations for each floor of each building in the simulation.
   - Uses the [Urbano plug-in](https://urbano.io) to import GIS geometries and metadata.  
   - Incorporates detected WWR values for each orientation of each building.
   - Required inputs: SHP files with building geometry and metadata, one for each archetype being simulated, plus one or more SHP files with context buildings (which are not simulated but act as shading objects).
   - File included is a Grasshopper binary (.gh) file.
