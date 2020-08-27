// Snow Metrics 2020
// Written and edited by Ryan L. Crumley in 2019 and Eugene Mar in 2018

Map.setOptions('Satellite');      

//////////////////////////////////////////////
////////   USER INPUT REQUIRED  //////////////
//////////////////////////////////////////////

// Change the year only and run
var wyr = 2016;


// Set the month and day of year for the Water Year to begin
var wyStartDay = '-10-01';

var NDSI_threshold = '15'

// Set the lower end of the SCF values for masking
// This is OPTIONAL. Setting this value to 0 will not mask any values.
// This value equals about 30 days of snow or less that is being masked. 
// This is set because without it there are 'snow' designations all across the southern U.S., Mexico, Central America, Africa, etc.
// My guess is that agricultural water bodies are being mistaken for snow, rice growing, flooded fields, and etc.
var low_value = 0.07;

// Set the prior_days parameter for the SDD calculation.
// This gets fed into the 'min_sdays_prior_to_SDD' image and variable.
// Setting this value means there are [prior_days] days that snow must be seen/designated in the pixel before the SDD date can be set.
var prior_days = 5;

// Set the parameter below for the SDD calculation.
// This gets fed into the 'minNoSnow' image and variable.
// Setting this value means there are [minNoSnowdays] days that snow must be absent from the pixel after the SDD date is set.
var minNoSnowdays = 5;

// Set the projection for the geotiff export
var projection = 'EPSG:4326';

// These are the min and max corners of the domain for export, in Lat, Long
// Input the minimum lat, lower left corner
var minLat = 28.0184;
// Input the minimum long, lower left corner
var minLong = -126.5330;
// Input the max lat, upper right corner
var maxLat = 50.1380;
// Input the max Long, upper right corner
var maxLong = -94.6287;

// My domain below is the western US and southern Canadian Rockies region
var my_domain = /* color: #0b4a8b */ee.Geometry.Polygon(
        [[[-126.533, 50.138],
          [-126.533, 28.0184],
          [-94.6287, 28.0184],
          [-94.6287, 50.138]]]);

// Uncomment the lines below to add the my_domain variable to the visualization
// Map.addLayer(my_domain,'','My Domain');
// print (my_domain);

//% These are the color palettes for the visualization of the metrics below.
var palette_snow = '081d58,253494,225ea8,1d91c0,41b6c4,7fcdbb,c7e9b4,edf8b1,ffffd9,ffffff';
var palette_sdd = '8c510a,bf812d,dfc27d,f6e8c3,f5f5f5,c7eae5,80cdc1,35978f,01665e';

//////////////////////////////////////////////
////////    END USER INPUT      //////////////
//////////////////////////////////////////////


// Call the getMetrics variable to run the function on the water year defined above.
var getMetrics = function(wyr){
    
    // Take the end of the water year and create a start and end date.
    var start = (wyr - 1) + wyStartDay;
    print(start,'start');
    var end = wyr + wyStartDay;

    // Take the end date of the water year and increase it by 30 days.
    var start_adv = end;
    var end_adv = ee.Date(end).advance(30,'day');
    print(start_adv, 'start_adv')
    print(end_adv,'end_adv');
    
    // Advance the days of interest by 30 days into the future
    var adv_days = ee.ImageCollection("MODIS/006/MOD10A1")
      .filterDate(start_adv,end_adv)
      .size().getInfo();
    print(adv_days,'adv_days')
    
    
    /////////////// ACCESS the DATA and do some things  //////////////////////////
    ///////////////////////////  SATELLITE DATASETS  /////////////////////////////
    //////////////////////////////////////////////////////////////////////////////

    
    // This makes a list of images where the MOD10A1v6 designation equals cloud(250).
    // See printout to console for more info.
    // This creates a yes/no (0/1) image of cloud locations each day of the year.
    var MODIS_cloud_list = ee.ImageCollection("MODIS/006/MOD10A1")
      .filterDate(start,end_adv)
      .map(function(img) {
        return img.expression("(BAND==250)",{BAND:img.select('NDSI_Snow_Cover_Class')})})
      .toList(400);
      print(MODIS_cloud_list,'MODIS_cloud_list');
    
    // This makes a list of images where the snow value is between 1 and 100 based on the snow 
    // cover band. This creates an yes/no (0/1) image of snow locations each day of the year.
    // See printout to console for more info.
    var MODIS_snow_list = ee.ImageCollection("MODIS/006/MOD10A1")
      .filterDate(start,end_adv)
      .map(function(img) {
        return img.expression('(BAND>='+ NDSI_threshold +'&&BAND<=100)',{BAND:img.select('NDSI_Snow_Cover')})})
      .toList(400);
      print(MODIS_snow_list,'MODIS_snow_list');
      
    // Access the 30m Landsat datset for water loctions. I don't want these metrics
    // to be calculated over inland water bodies!
    // See printout to console for more info.
    var water = ee.ImageCollection("GLCF/GLS_WATER")
        .map(function(img) {
          return img.expression("(BAND==2)",{BAND:img.select('water')})}).or().not()
    print(water,'This is water from LandSat')
    // Map.addLayer(water,{'palette':palette_snow,'min':0, 'max':1}, 'Inland Water 0=water, 1=not water');
    
    
    // Get number of days in the Water Year
    var ndays = MODIS_snow_list.length().getInfo() - adv_days;
    print(ndays,'ndays');
    
    //% This is a correction for leap years.
    var sddCorrection = (ndays < 365 + (wyr % 4 ? 0 : 1)) ? 365 + (wyr % 4 ? 0 : 1) - ndays : 0;
    print('Processed ' + ndays + ' days in ' + wyr);
    
    ///////////////////////////////////////////////////////////////////////////
    // Open a bunch of image variables with zero values, or other defined values
    
    var scfImage = 'MODIS_SCF_' + wyr;
    print(scfImage,'scfImage');
    var sddImage = 'MODIS_SDD_' + wyr;
    print(sddImage,'sddImage');
    sddImage = ee.Image(0);
    
    var future_snow = ee.Image(0);
    var Snow = ee.Image(0);
    var accSnow = ee.Image(0);
    var ephemeral_counter = ee.Image(0);
    var noSnowCounter = ee.Image(0);
    var minNoSnow = ee.Image(minNoSnowdays);
    var min_sdays_prior_to_SDD = ee.Image(prior_days);
    var sddDetected = ee.Image(0);
    var MODIS_max_ndays_nosnow = ee.Image(0);
    
    //////////////////////////////////////////////////////////
    ///////////////  Do The STUFFFFFF  ///////////////////////
    //////////////////////////////////////////////////////////
    // Increment backwards from the end of the water year + 30 days
    // REMEMBER: This is complex because it is moving backwards in time. Its really easy to forget. 
    // All the commenting is interpreted through the backwards-through-time lens.
    for(var current_day =ndays + adv_days-1; current_day>=0; current_day--) {
      
      // During those 30 days after the water year, build some 'memory' of where snow is and is not in the variables
      // Take cloud locations from MODIS and create cloud image of 1=cloud, 0=everything else
      var Cloud = ee.Image(MODIS_cloud_list.get(current_day)).unmask();
      
      // If there is a cloud designation TODAY and future_snow variable=1, from tomorrow or snow=1 tomorrow, set to 1. 
      // Essentially, 'remembering' where snow is located in the future
      future_snow = Cloud.and(future_snow.or(Snow));
      
      // Take snow locations from MODIS TODAY and create snow image of 1=snow, 0=everything else for TODAY
      Snow = ee.Image(MODIS_snow_list.get(current_day)).unmask();
      
      // If TODAY is in the water year of interest do all these things
      // Because of advanced days process above, we have snow 'memory' of the future
      if (current_day < ndays) {
        
        // If TODAY is snow in the pixel or if there is future snow in the pixel, snow_occurrence = 1 
        var snow_occurrence = future_snow.or(Snow);
        
        // Count the number of days snow has occurred in the future in the pixel and then accumulate it to this variable.
        // This is the last time we use accsnow to create the final SCF_image
        accSnow = accSnow.add(snow_occurrence);
        
        ////////////////////////////////////////////////////////////////////
        // Everything below in the for loop is to deal with the SDD issues.
        // This accesses the variable defined as prior_days and creates a conditional statement
        // If prior_days is greater than the ephemeral counter, and snow does not occur in the future, reset the value of this variable to ephemeral counter value
        // Purpose:
        min_sdays_prior_to_SDD = min_sdays_prior_to_SDD.where(snow_occurrence.eq(0).and(ephemeral_counter.gt(min_sdays_prior_to_SDD)), ephemeral_counter);
        
        // The number of days after the the TODAY that has snow_occurrence=1 (snow or previously snow)
        ephemeral_counter = (ephemeral_counter.add(snow_occurrence)).multiply(snow_occurrence);
        
        // A running count of the number of days when snow_occurrence was equal to 0 from end of water year to now
        // SDD cannot occur until some number of days without snow (minNoSnow) happen at the end of the year
        noSnowCounter = (noSnowCounter.add(snow_occurrence.eq(0))).multiply(sddDetected.eq(0));
        
        // Day of snow disappearance
        // Number of days of snow prior to CURRENT day must excede a value set by prior_days variable
        // And there must be a minimum number of days without snow after the CUURENT day (minNoSnow)
        sddDetected = (ephemeral_counter.gt(min_sdays_prior_to_SDD)).and(noSnowCounter.gt(minNoSnow));
        
        
        minNoSnow = minNoSnow.where(sddDetected, noSnowCounter);
        
        // Find the day of snow disappearance by adding some values to the CURRENT day
        var sdd1 = current_day+sddCorrection;
        var sdd = min_sdays_prior_to_SDD.add(ee.Image(sdd1));
        sddImage = sddImage.where(sddDetected, sdd);
        
      }
    }
    
    sddImage = sddImage.where(accSnow.eq(ndays-1), ndays+sddCorrection);
    
    
    // Make the final scf image by dividing the number of snow days by the number of days in the year
    // Also, mask out all the inland water bodies using the Landsat dataset (its more precise than the MODIS land mask, for sure)
    scfImage = accSnow.divide(ndays).updateMask(accSnow).rename('ccSCF').updateMask(water);
    print(scfImage);
    
    // Make the final scf image
    // Also, mask out all the inland water bodies
    sddImage = sddImage.updateMask(sddImage).rename('SDD').updateMask(water);
    print(sddImage);
    
    ////////////////////////////////////////////////////
    /////////       Mask the images         ////////////
    ////////////////////////////////////////////////////
    // Mask the low_values in SCF.
    var scfimg = scfImage;
    print(scfimg, 'SCF');
    var low_end = ee.Image(low_value);
    var lower_mask = scfimg.gte(low_end);
    var SCF = scfimg.updateMask(lower_mask);
    Map.addLayer(SCF,{'palette':palette_snow,'min':low_value, 'max':1},'SCF (post-mask)');
    
    // Mask the SDD image by the low_values image above, just so we have the same coverage
    var sddimg = sddImage;
    var SDD = sddimg.updateMask(lower_mask);
    Map.addLayer(SDD,{'palette':palette_sdd,'min':0, 'max':365},'SDD (post-mask)');
    
    
    //////////////////////////////////////// 
    /////////  Export everything ///////////
    ////////////////////////////////////////
    
    // Uncomment below for global water year images
    // To Asset, only relevant for the global water year images for the App
    // Export.image.toAsset({
    // image: SCF,
    // description: 'SCF'+ wyr + 'global_toAsset',
    // assetId: 'users/ryanlcrumley/metrics/wyr/'+'SCF'+wyr+'global_15',
    // scale: 500,
    // crs: projection,
    // //region: my_domain,
    // region: global_extent,
    // maxPixels:1e12,
    // });
    
    Export.image.toDrive({
    image: SCF,
    description: 'SCF'+wyr+'global',
    //assetId: 'users/ryanlcrumley/metrics/wyr/'+'SCF'+wyr+'global',
    scale: 2000,
    crs: projection,
    //region: my_domain,
    region: global_extent,
    maxPixels:1e12,
    });
    
    // Export.image.toAsset({
    // image: SDD,
    // description: 'SDD'+ wyr + 'global_toAsset',
    // assetId: 'users/ryanlcrumley/metrics/wyr/'+'SDD'+wyr+'global_15',
    // scale: 500,
    // crs: projection,
    // //region: my_domain,
    // region: global_extent,
    // maxPixels:1e12,
    // });
    
    Export.image.toDrive({
    image: SDD,
    description: 'SDD'+wyr+'global',
    scale: 2000,
    crs: projection,
    region: global_extent,
    maxPixels:1e12,
    });
    
    
    // // Uncomment below for western_US or any other user defined area
    // // Export your domain of interest to your drive
    // Export.image.toDrive({
    // image: SCF,
    // description: 'SCF'+ wyr + 'western_US',
    // scale: 2000,
    // crs: projection,
    // region: my_domain,
    // maxPixels:1e12,
    // });

    // Export.image.toDrive({
    // image: SDD,
    // description: 'SDD'+ wyr + 'western_US',
    // scale: 2000,
    // crs: projection,
    // region: my_domain,
    // maxPixels:1e12,
    // });
    
};

getMetrics(wyr);


// Define the global extent variable for global-scale export only
var global_extent = 
    /* color: #0b4a8b */
    /* shown: false */
    ee.Geometry.Polygon(
        [[[-162.0128416755419, 72.65326131729866],
          [-154.6300291755419, 72.54813418931167],
          [-143.0284666755419, 71.68462403627241],
          [-130.8995604255419, 71.62930475254494],
          [-131.2511229255419, 76.99515967718128],
          [-100.31362292554189, 82.50352619046451],
          [-87.83315417554189, 83.32648463709445],
          [-68.84877917554189, 83.43790136386835],
          [-58.74135730054189, 82.81785104499237],
          [-59.88393542554189, 82.27067001612757],
          [-62.87221667554189, 81.7327790132784],
          [-63.66323230054189, 81.35809863032443],
          [-68.23354480054189, 80.54303104998446],
          [-69.02456042554189, 79.90143318371658],
          [-73.15541980054189, 79.01766966080187],
          [-74.91323230054189, 77.40405034834552],
          [-71.92495105054189, 75.732072721478],
          [-62.6685857556447, 73.87555651752812],
          [-57.2857760540478, 70.34812340893819],
          [-55.3082369915478, 65.05335821275636],
          [-54.6490573040478, 61.83514086304732],
          [-50.9576510540478, 58.42442746250207],
          [-40.0152682415478, 58.813442770774536],
          [-37.5543307415478, 60.629818625739745],
          [-36.023875884494714, 63.12787449030399],
          [-17.039500884494714, 68.5710829809486],
          [10.382374115505286, 72.7466890038922],
          [2.6479991155052858, 80.55132546788323],
          [44.483936615505286, 81.68937693871335],
          [99.86345504743895, 81.66585473479131],
          [119.81462692243895, 78.14323874412803],
          [152.50993942243883, 77.81375570127527],
          [177.54253498554283, 73.84118739234809],
          [-174.89887126445717, 72.83343167245718],
          [-171.55902751445717, 68.45311145590779],
          [-169.09809001445717, 66.7457599665082],
          [-169.36176188945717, 65.71865915572815],
          [-170.85590251445717, 65.20769048959903],
          [-169.62543376445717, 64.46024674589395],
          [-168.13129313945717, 63.885921148885686],
          [-168.30707438945717, 62.98193808381884],
          [-171.20746501445717, 60.05031132706765],
          [-173.31684001445717, 57.64158142688992],
          [-172.14643793571543, 52.74541365183963],
          [-172.05854731071543, 51.32629672441272],
          [-175.916139384933, 46.85291107176064],
          [177.05261061506712, 42.35067672536996],
          [159.47448561506712, 9.17836066284905],
          [178.81042311506712, -17.234926345454785],
          [-174.158326884933, -43.26895835514316],
          [172.83386061506712, -54.93325871028938],
          [147.52136061506712, -58.22263172682042],
          [117.28698561506712, -52.22086095965297],
          [102.87292311506712, -28.545626662467708],
          [92.32604811506712, -9.372856733411277],
          [70.52917311506712, -4.8388677677117835],
          [65.60729811506712, -25.095240229817463],
          [42.755735615067124, -39.30853687705388],
          [1.974485615067124, -40.65538046319251],
          [-32.47863938493293, -42.236399821456985],
          [-48.29895188493293, -50.23996114434759],
          [-54.27551438493293, -60.89287908714532],
          [-82.75207688493293, -59.495130440761535],
          [-82.40051438493293, -50.46428123885867],
          [-77.12707688493293, -36.53584908843394],
          [-83.10363938493293, -18.573039167986042],
          [-99.97863938493293, 6.042574667239055],
          [-118.78723313493293, 22.177547053420877],
          [-130.21301438493293, 41.95974326084755],
          [-134.60754563493293, 52.63660326691857],
          [-147.43957688493293, 56.310632014370825],
          [-164.66613938493293, 49.660737101241025],
          [-171.32302382389855, 51.905514686008644],
          [-171.73834900357065, 53.59556816916435],
          [-171.82623962857065, 57.518594826381026],
          [-168.75006775357065, 60.93293992531976],
          [-166.83118659964782, 63.804172102156706],
          [-168.80872566214782, 65.22205721986592],
          [-167.66614753714782, 66.8285513233769],
          [-168.54505378714782, 71.40081086129041]]]);
