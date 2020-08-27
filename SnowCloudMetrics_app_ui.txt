// GEE IMPORTS
// If using this script in the Chrome-based GEE code editor,
// GEE will display a prompt to convert these declarations
// into imports. We recommend you do so.
var countries = ee.FeatureCollection("USDOS/LSIB_SIMPLE/2017"),
    states = ee.FeatureCollection("TIGER/2018/States"),
    amuDarya = ee.FeatureCollection("users/rpalomaki/scm_app/amu_darya"),
    provinces = ee.FeatureCollection("FAO/GAUL_SIMPLIFIED_500m/2015/level1"),
    countryNames = ee.FeatureCollection("users/rpalomaki/scm_app/country_list"),
    provinceNames = ee.FeatureCollection("users/rpalomaki/scm_app/province_list"),
    stateNames = ee.FeatureCollection("users/rpalomaki/scm_app/states_list"),
    huc04 = ee.FeatureCollection("USGS/WBD/2017/HUC04"),
    huc06 = ee.FeatureCollection("USGS/WBD/2017/HUC06"),
    huc08 = ee.FeatureCollection("USGS/WBD/2017/HUC08"),
    huc10 = ee.FeatureCollection("USGS/WBD/2017/HUC10"),
    snotel = ee.FeatureCollection("users/ryanlcrumley/vector/snotel_points"),
    srtm = ee.Image("USGS/SRTMGL1_003"),
    waterMaskIC = ee.ImageCollection("MODIS/006/MOD44W"),
    SCF_IC = ee.ImageCollection("users/rpalomaki/scm_app/SCF_IC_v2"),
    SDD_IC = ee.ImageCollection("users/rpalomaki/scm_app/SDD_IC_v2");

//////////////////////////////////////////////////////////
//////////////////////////////////////////////////////////
//      SnowCloudMetrics - Google Earth Engine App      //
//////////////////////////////////////////////////////////
//////////////////////////////////////////////////////////

// First, some housekeeping to add a property to SCF/SDD images.
// This is necessary for the timeseries charts later.
// To do: add this property to the individual images before saving IC as asset!

var scfList = SCF_IC.toList(19);
var test = [];
test.push(ee.Image(scfList.get(0)).set({'system:time_start': ee.Date('2000-10-01')}));
test.push(ee.Image(scfList.get(1)).set({'system:time_start': ee.Date('2001-10-01')}));
test.push(ee.Image(scfList.get(2)).set({'system:time_start': ee.Date('2002-10-01')}));
test.push(ee.Image(scfList.get(3)).set({'system:time_start': ee.Date('2003-10-01')}));
test.push(ee.Image(scfList.get(4)).set({'system:time_start': ee.Date('2004-10-01')}));
test.push(ee.Image(scfList.get(5)).set({'system:time_start': ee.Date('2005-10-01')}));
test.push(ee.Image(scfList.get(6)).set({'system:time_start': ee.Date('2006-10-01')}));
test.push(ee.Image(scfList.get(7)).set({'system:time_start': ee.Date('2007-10-01')}));
test.push(ee.Image(scfList.get(8)).set({'system:time_start': ee.Date('2008-10-01')}));
test.push(ee.Image(scfList.get(9)).set({'system:time_start': ee.Date('2009-10-01')}));
test.push(ee.Image(scfList.get(10)).set({'system:time_start': ee.Date('2010-10-01')}));
test.push(ee.Image(scfList.get(11)).set({'system:time_start': ee.Date('2011-10-01')}));
test.push(ee.Image(scfList.get(12)).set({'system:time_start': ee.Date('2012-10-01')}));
test.push(ee.Image(scfList.get(13)).set({'system:time_start': ee.Date('2013-10-01')}));
test.push(ee.Image(scfList.get(14)).set({'system:time_start': ee.Date('2014-10-01')}));
test.push(ee.Image(scfList.get(15)).set({'system:time_start': ee.Date('2015-10-01')}));
test.push(ee.Image(scfList.get(16)).set({'system:time_start': ee.Date('2016-10-01')}));
test.push(ee.Image(scfList.get(17)).set({'system:time_start': ee.Date('2017-10-01')}));
test.push(ee.Image(scfList.get(18)).set({'system:time_start': ee.Date('2018-10-01')}));

SCF_IC = ee.ImageCollection(test);

var sddList = SDD_IC.toList(19);
test = [];
test.push(ee.Image(sddList.get(0)).set({'system:time_start': ee.Date('2000-10-01')}));
test.push(ee.Image(sddList.get(1)).set({'system:time_start': ee.Date('2001-10-01')}));
test.push(ee.Image(sddList.get(2)).set({'system:time_start': ee.Date('2002-10-01')}));
test.push(ee.Image(sddList.get(3)).set({'system:time_start': ee.Date('2003-10-01')}));
test.push(ee.Image(sddList.get(4)).set({'system:time_start': ee.Date('2004-10-01')}));
test.push(ee.Image(sddList.get(5)).set({'system:time_start': ee.Date('2005-10-01')}));
test.push(ee.Image(sddList.get(6)).set({'system:time_start': ee.Date('2006-10-01')}));
test.push(ee.Image(sddList.get(7)).set({'system:time_start': ee.Date('2007-10-01')}));
test.push(ee.Image(sddList.get(8)).set({'system:time_start': ee.Date('2008-10-01')}));
test.push(ee.Image(sddList.get(9)).set({'system:time_start': ee.Date('2009-10-01')}));
test.push(ee.Image(sddList.get(10)).set({'system:time_start': ee.Date('2010-10-01')}));
test.push(ee.Image(sddList.get(11)).set({'system:time_start': ee.Date('2011-10-01')}));
test.push(ee.Image(sddList.get(12)).set({'system:time_start': ee.Date('2012-10-01')}));
test.push(ee.Image(sddList.get(13)).set({'system:time_start': ee.Date('2013-10-01')}));
test.push(ee.Image(sddList.get(14)).set({'system:time_start': ee.Date('2014-10-01')}));
test.push(ee.Image(sddList.get(15)).set({'system:time_start': ee.Date('2015-10-01')}));
test.push(ee.Image(sddList.get(16)).set({'system:time_start': ee.Date('2016-10-01')}));
test.push(ee.Image(sddList.get(17)).set({'system:time_start': ee.Date('2017-10-01')}));
test.push(ee.Image(sddList.get(18)).set({'system:time_start': ee.Date('2018-10-01')}));

SDD_IC = ee.ImageCollection(test);


// Now on to the actual app...

/////////////////////////////////////////////
//      Initial variable declarations      //
/////////////////////////////////////////////

// Initialize the map.
// ===================
var map = ui.Map();
map.drawingTools().setShown(false);
map.drawingTools().setDrawModes(['polygon','rectangle']);
map.drawingTools().setLinked(false);
map.setOptions('Satellite');
var snotelLayer = ui.Map.Layer({
  eeObject: snotel.draw({pointRadius: 5, color: 'dd1cff'}), // or try 6fff1c
  visParams: null, // no visParams because of .draw()
  name: 'SNOTEL sites'});
map.add(snotelLayer);
map.layers().get(0).setShown(false);


// Define various parameters that need global scope.
// =================================================
// Palettes for SCF and SDD
var palette_scf = '081d58,253494,225ea8,1d91c0,41b6c4,7fcdbb,c7e9b4,edf8b1,ffffd9,ffffff';
var palette_sdd = '8c510a,bf812d,dfc27d,f6e8c3,f5f5f5,c7eae5,80cdc1,35978f,01665e';
// UI Panels
var waterYearDropdownPanel;
var geoDropdownPanel;
var geoPanel2;
var elevationFilterCheckbox;
var elevationFilterPanel;
var regionDetailsPanel;
var downloadButtonPanel;
var pointDetailsPanel;
var chartPanel;
// SCF and SDD need global scope for download URL
var SCF;
var SDD;
var urlGeo;
// Some variables for timeseries visualization
var pointColors = ['red','cyan','magenta','yellow','orange', 'green', 'black','purple'];
var pointColorCounter = 0;
var pointColor;
var pointColorDisplay;
var pointNameCounter = 1;
// Select most recent year for MODIS water mask.
var waterMaskList = waterMaskIC.toList(16);
var waterMask = ee.Image(waterMaskList.get(15)).select('water_mask');

////////////////////////////////
//      Helper functions      //
////////////////////////////////

// Create and add SCF/SDD colorbar legends to map.
// ===============================================
var SCFpaletteLabels = [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0];
var SDDpaletteLabels = [0, 40, 80, 120, 160, 200, 240, 280, 320, 365];


// Create colorbar.
// ================
function makeColorBar(palette, min, max) {
  return ui.Thumbnail({
    image: ee.Image.pixelLonLat().select(0),
    params: {
      bbox: [0, 0, 1, 0.1],
      dimensions: '200x15',
      format: 'png',
      min: min,
      max: max,
      palette: palette,
    },
    style: {stretch: 'horizontal', margin: '0px 8px', position: 'bottom-left'},
  });
}


// Create labeled colorbar legend.
// ===============================
function makeLegend(title, labels, palette, min, max, position) {
  var labelPanel = ui.Panel();
  for (var i = 0; i < labels.length; i++) {
    var label = ui.Label({
      value: labels[i],
      style: {
        maxWidth: '25px',
        margin: '4px 8px',
        position: 'bottom-left'
      }
    });
    labelPanel.add(label);
  }
  labelPanel.setLayout(ui.Panel.Layout.flow('horizontal'));

  var titleLabel = ui.Label({
    value: title,
    style: {
      fontSize: '14px',
      fontWeight: 'bold',
      position: 'top-center'
    }
  });

  return ui.Panel({
    widgets: [titleLabel, makeColorBar(palette, min, max), labelPanel],
    style: {position: position}
  });
}

// Add colorbars, default hidden.
map.add(makeLegend('Snow Cover Frequency', SCFpaletteLabels, palette_scf, 0, 1, 'bottom-left'));
map.add(makeLegend('Snow Disappearance Date [DoWY]', SDDpaletteLabels, palette_sdd, 0, 1, 'bottom-right'));
map.widgets().get(0).style().set('shown', false);
map.widgets().get(1).style().set('shown', false);


// Colorbar checkboxes for main panel.
// ===================================
var makeColorbarCheckboxes = function() {
  var SCFbox = ui.Checkbox({
    label: 'Display SCF legend',
    onChange: function(checked) {
      map.widgets().get(0).style().set('shown', checked); // SCF added to map first --> index 0
    }
  });
  var SDDbox = ui.Checkbox({
    label: 'Display SDD legend',
    onChange: function(checked) {
      map.widgets().get(1).style().set('shown', checked); // SDD added to map second --> index 1
    }
  });

  return ui.Panel([SCFbox, SDDbox], ui.Panel.Layout.flow('horizontal'));
};


// Remove specified layer from map.
// ================================
function removeMapLayer(layerName) {
  var layers = map.layers();
  var layerNames = [];
  layers.forEach(function(lay) {
    var layName = lay.getName();
    layerNames.push(layName);
  });
  var ind = layerNames.indexOf(layerName);
  if (ind > -1) {
    map.remove(layers.get(ind))}
}


// Select all point layers for timseries charts.
// =============================================
function collectPointsOnMap() {
  var pointGeo;
  var layers = map.layers();
  // Check all layer names for the string 'Point'
  var allLayers = [];
  var pointLayers = [];
  layers.forEach(function(lay) {
    var layName = lay.getName();
    allLayers.push(layName); // necessary for indexing below
    if (layName.indexOf('Point') !== -1) {
      pointLayers.push(layName);
    }
  });
  // Create list for future FeatureCollection
  var pointFeatures = [];
  // Get geometries (as eeObjects) from all point layers
  pointLayers.forEach(function(pointLay) {
    var ind = allLayers.indexOf(pointLay);
    var pointGeo = layers.get(ind).getEeObject().geometry();
    var pointFeature = ee.Feature(pointGeo, {label: pointLay});
    pointFeatures.push(pointFeature);
  });

  var pointsFC = ee.FeatureCollection(pointFeatures);
  return pointsFC;
}


// Remove all points from map.
// ===========================
function removePointsFromMap() {
  var layers = map.layers();
  // Check all layer names for the string 'Point'
  var pointLayers = [];
  layers.forEach(function(lay) {
    var layName = lay.getName();
    if (layName.indexOf('Point') !== -1) {
      pointLayers.push(layName);
    }
  });
  // Remove point layers
  pointLayers.forEach(function(lay) {
    layers = map.layers(); // need to update after each removal
    var layerNames = [];
    layers.forEach(function(layer) {
      var layName = layer.getName();
      layerNames.push(layName);
    });
    var ind = layerNames.indexOf(lay);
    map.remove(layers.get(ind));
  });
}

////////////////////////////////////////////////
////////////////////////////////////////////////
//      UI elements - panels and widgets      //
////////////////////////////////////////////////
////////////////////////////////////////////////


/////////////////////////////////////////////
//      Title panel with reset button      //
/////////////////////////////////////////////

// Create title and reset button.
// ==============================
function makeTitleAndReset() {
  var title = ui.Label({
    value: 'SnowCloudMetrics App',
    style: {stretch: 'vertical',
    fontSize: '18px',
    fontWeight: '100',
    padding: '5px'}
  });
  var resetButton = ui.Button({
    label: 'Reset',
    onClick: function() {
      map.clear();
      map.drawingTools().setShown(false);
      map.drawingTools().clear();
      map.addLayer(snotel.draw({pointRadius: 5, color: 'dd1cff'}), null, 'SNOTEL sites', false);
      snotelCheckbox.setValue(false);
      waterYearDropdownPanel.clear();
      geoDropdownPanel.clear();
      geoPanel2.clear();
      pointDetailsPanel.clear();
      chartPanel.clear();

      waterYearDropdownPanel.add(makeWaterYearDropdown());
      geoDropdownPanel.add(makeGeometryDropdown());
      regionDetailsPanel.style().set('shown', false);
      downloadButtonPanel.style().set('shown', false);
      pointDetailsPanel.add(pointDetails);
      pointDetailsPanel.add(makeTimeseriesButtons());
      pointDetailsPanel.style().set('shown', false);

      map.onClick(function(coordinates) {
        if (waterYearDropdown.getValue() < 10) {
          alert('Please select a water year to display SCF and SDD data at your selected point.');
        }
        else {
          var point = ee.Geometry.Point([coordinates.lon, coordinates.lat]);
          getPointDetails(point);
        }
      });
    }
  });

  return ui.Panel([title, resetButton], ui.Panel.Layout.flow('horizontal'));
}


// Checkbox for SNOTEL layer.
// ==========================
var snotelCheckbox = ui.Checkbox({
  label: 'Display SNOTEL sites',
  value: false,
  onChange: function(checked) {
    var layers = map.layers();
    var layerNames = [];
    layers.forEach(function(lay) {
      var layName = lay.getName();
      layerNames.push(layName)});
    var ind = layerNames.indexOf('SNOTEL sites');
    map.layers().get(ind).setShown(checked);
  }
});


/////////////////////////////////////////////////
//      Data selection for SCF/SDD images      //
/////////////////////////////////////////////////

// Water year dropdown selection for SCF/SDD data.
// ===============================================
var waterYearDropdown;
function makeWaterYearDropdown() {
  var years = ['2001','2002','2003','2004','2005','2006','2007','2008','2009',
               '2010','2011','2012','2013','2014','2015','2016','2017','2018','2019'];
  waterYearDropdown = ui.Select({
    items: years,
    placeholder: 'Select a water year...',
    onChange: function(year) {
      waterYearDropdown.setValue(year);
    }
  });
  return ui.Panel(waterYearDropdown);
}


////////////////////////////////
//      Elevation filter      //
////////////////////////////////

// Instructions and textboxes for min/max elevation.
// =================================================
var minElevationTextbox;
var maxElevationTextbox;
var makeElevationFilter = function() {
  var elevationInstructions = ui.Label({
    value: 'Enter minimum and/or maximum elevation in meters.\nOriginal region will be displayed with masked data.',
    style: {
      color: 'black',
      fontWeight: '100',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });
  var minElevationLabel = ui.Label({
    value: 'Minimum:',
    style: {stretch: 'vertical', fontWeight: '100'}
  });
  minElevationTextbox = ui.Textbox({
    placeholder: 'Min elevation',
    style: {width: '100px'},
    onChange: function(text) {
      minElevationTextbox.setValue(text);
    }
  });

  var maxElevationLabel = ui.Label({
    value: 'Maximum:',
    style: {stretch: 'vertical', fontWeight: '100'}
  });
  maxElevationTextbox = ui.Textbox({
    placeholder: 'Max elevation',
    style: {width: '100px'},
    onChange: function(text) {
      maxElevationTextbox.setValue(text);
    }
  });

  var filter =  ui.Panel([minElevationLabel, minElevationTextbox,
                          maxElevationLabel, maxElevationTextbox],
                          ui.Panel.Layout.flow('horizontal'));
  return ui.Panel([elevationInstructions, filter], ui.Panel.Layout.flow('vertical'));
};


//////////////////////////////////
//      Geometry selection      //
//////////////////////////////////

// Initialize label for region-averaged SCF/SDD.
// =============================================
var regionDetails = ui.Label({
  value: 'Region-averaged metrics: ',
  style: {
    fontSize: '14px',
    fontWeight: '100',
    padding: '5px',
    whiteSpace: 'pre'
  }
});


// First dropdown menu for geometry selection.
// ===========================================
var newGeo; //needs global scope; put here to accomodate Amu Darya
function makeGeometryDropdown() {
  var dropdownItems = ['Country','US State','Canadian Province','Amu Darya watershed',
                       'USGS HUC','Bounding Box','Draw a Polygon','Use my own GEE asset'];
  var geometryDropdown = ui.Select({
    items: dropdownItems,
    onChange: function(key) {
      regionDetailsPanel.style().set('shown', false);
      // Create a second dropdown or a textbox in geoPanel2 depending on geometry.
      switch (key) {
        case 'Country':
          geoPanel2.add(makeSecondGeoDropdown('Country'));
          break;
        case 'US State':
          geoPanel2.add(makeSecondGeoDropdown('US State'));
          break;
        case 'Canadian Province':
          geoPanel2.add(makeSecondGeoDropdown('Canadian Province'));
          break;
        case 'Amu Darya watershed':
          geoPanel2.add(makeAmuDaryaPanel());
          break;
        case 'USGS HUC':
          geoPanel2.add(makeGeoTextbox('USGS HUC'));
          break;
        case 'Bounding Box':
          geoPanel2.add(makeGeoTextbox('Bounding Box'));
          break;
        case 'Draw a Polygon':
          geoPanel2.add(userDrawnPolygon());
          break;
        case 'Use my own GEE asset':
          geoPanel2.add(makeGeoTextbox('User Asset'));
      }
    }
  });

  geometryDropdown.setPlaceholder('Choose a geometry option...');
  return ui.Panel(geometryDropdown);
}


// Second geometry dropdown for Countries, States, Provinces.
// ==========================================================
function makeSecondGeoDropdown(key) {
  var geoType = key;
  var geoList;
  var geoNameProperty;
  var nameList;
  var dropdownItems;
  var placeholder;
  var newGeoList;
  // Clear and turn off drawing tools.
  map.drawingTools().clear();
  map.drawingTools().setShown(false);

  // Set appropriate variables based on selection.
  switch (key) {
    case 'Country':
      geoList = countries;
      geoNameProperty = 'country_na';
      nameList = countryNames.toList(285);
      dropdownItems = nameList.map(function(el){
        return ee.Feature(el).get(ee.Feature(nameList.get(0)).propertyNames().get(0));
      }).sort();
      placeholder = 'Select a country...';
      break;
    case 'US State':
      geoList = states;
      geoNameProperty = 'NAME';
      nameList = stateNames.toList(50);
      dropdownItems = nameList.map(function(el){
        return ee.Feature(el).get(ee.Feature(nameList.get(0)).propertyNames().get(0));
      }).sort();
      placeholder = 'Select a state...';
      break;
    case 'Canadian Province':
      geoList = provinces.filter(ee.Filter.eq('ADM0_NAME','Canada'));
      geoNameProperty = 'ADM1_NAME';
      nameList = provinceNames.toList(20);
      dropdownItems = nameList.map(function(el){
        return ee.Feature(el).get(ee.Feature(nameList.get(0)).propertyNames().get(0));
      }).sort();
      placeholder = 'Select a province...';
  }

  // After setting variables, construct dropdown widget.
  var geoDropdown2 = ui.Select({
    items: dropdownItems.getInfo(),
    onChange: function(key2) {
      urlGeo = key2.replace(/\s/g,'');
      if (geoNameProperty == 'ADM1_NAME') {
        // Different filter for Canadian provinces.
        newGeoList = geoList.filter(ee.Filter.stringContains(geoNameProperty, key2)).toList(50);
      }
      else {
        // Countries and US States can use ee.Filter.eq.
        newGeoList = geoList.filter(ee.Filter.eq(geoNameProperty, key2)).toList(50);
      }

      if (newGeoList.size().getInfo() > 1) {
        // Take the union of the geometries if necessary.
        newGeo = ee.FeatureCollection(newGeoList).union().geometry();
      }
      else {
        // No union, just take the first (and only) element of the list.
        newGeo = ee.Feature(newGeoList.get(0)).geometry();
      }
    }

  });

  // Submit button - adds geometry and data layers to map.
  var submit = ui.Button({
    label: 'Submit',
    onClick: function() {
      // Always add selected region to map.
      removeMapLayer('Selected region');
      removeMapLayer('SCF');
      removeMapLayer('SDD');
      map.addLayer(newGeo, null, 'Selected region', true, 0.5);
      map.centerObject(newGeo, 6);
      // Only add data layers if a water year is selected.
      var waterYear = waterYearDropdown.getValue();
      if (waterYear > 0) {
        removeMapLayer('SNOTEL sites');
        SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
        SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');

        // Elevation filter
        if (elevationFilterCheckbox.getValue()) {
          var elevMask;
          var srtmRegion = srtm.clip(newGeo);
          var minElev = parseFloat(minElevationTextbox.getValue());
          var maxElev = parseFloat(maxElevationTextbox.getValue());
          // Check for which numbers (min and/or max) were entered
          if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
            var minElevMask = srtmRegion.select('elevation').gte(minElev);
            var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
            elevMask = minElevMask.bitwiseAnd(maxElevMask);
          }
          else if (! isNaN(minElev)) { // number entered only for min elevation
            elevMask = srtm.select('elevation').gte(minElev);
          }
          else if (! isNaN(maxElev)) { // number only entered for max elevation
            elevMask = srtm.select('elevation').lte(maxElev);
          }
          // Update SCF and SDD with mask
          SCF = SCF.updateMask(elevMask);
          SDD = SDD.updateMask(elevMask);
        }
        map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
        map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
        map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

        // Calculate region-averaged metrics.
        regionDetailsPanel.style().set('shown', true);
        regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
        var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
        var SCFavgString = Number(SCFavg).toFixed(4);
        var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
        var SDDavgString = Number(SDDavg).toFixed(0);

        // Convert SDD day of water year integer to date.
        var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
        var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

        // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
        var newString = ee.String('Region-averaged metrics:')
                          .cat(ee.String('\n\tWater year: '))
                          .cat(ee.String(waterYearDropdown.getValue()))
                          .cat(ee.String('\n\tSCF: '))
                          .cat(ee.String(SCFavgString))
                          .cat(ee.String('\n\tSDD: '))
                          .cat(ee.String(SDDavgString))
                          .cat(ee.String(' ('))
                          .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

        newString.evaluate(function(val) {regionDetails.setValue(val)});
        // Display download button - not implemented in v1.0.
        //downloadButtonPanel.style().set('shown', true);
      }
    }
  });

  // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
  elevationFilterCheckbox = ui.Checkbox({
    value: false,
    label: 'Enable elevation filter',
    onChange: function(checked) {
      elevationFilterPanel.style().set('shown', checked);
    }
  });
  elevationFilterPanel = ui.Panel();
  elevationFilterPanel.add(makeElevationFilter());
  elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

  // Return widgets packaged in a panel.
  geoPanel2.clear();
  geoDropdown2.setPlaceholder(placeholder);
  return ui.Panel([geoDropdown2, elevationFilterCheckbox,
                   elevationFilterPanel, submit],
                  ui.Panel.Layout.flow('vertical'));
}


// Amu Darya watershed - no second dropdown, still need elev filter.
// =================================================================
function makeAmuDaryaPanel() {
  newGeo = amuDarya.geometry();
  // Submit button - adds geometry and data layers to map.
  var submit = ui.Button({
    label: 'Submit',
    onClick: function() {
      // Always add selected region to map.
      removeMapLayer('Selected region');
      removeMapLayer('SCF');
      removeMapLayer('SDD');
      map.addLayer(newGeo, null, 'Selected region', true, 0.5);
      map.centerObject(newGeo, 6);
      // Only add data layers if a water year is selected.
      var waterYear = waterYearDropdown.getValue();
      if (waterYear > 0) {
        removeMapLayer('SNOTEL sites');
        SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
        SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');

        // Elevation filter
        if (elevationFilterCheckbox.getValue()) {
          var elevMask;
          var srtmRegion = srtm.clip(newGeo);
          var minElev = parseFloat(minElevationTextbox.getValue());
          var maxElev = parseFloat(maxElevationTextbox.getValue());
          // Check for which numbers (min and/or max) were entered
          if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
            var minElevMask = srtmRegion.select('elevation').gte(minElev);
            var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
            elevMask = minElevMask.bitwiseAnd(maxElevMask);
          }
          else if (! isNaN(minElev)) { // number entered only for min elevation
            elevMask = srtm.select('elevation').gte(minElev);
          }
          else if (! isNaN(maxElev)) { // number only entered for max elevation
            elevMask = srtm.select('elevation').lte(maxElev);
          }
          // Update SCF and SDD with mask
          SCF = SCF.updateMask(elevMask);
          SDD = SDD.updateMask(elevMask);
        }
        map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
        map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
        map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

        // Calculate region-averaged metrics.
        regionDetailsPanel.style().set('shown', true);
        regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
        var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
        var SCFavgString = Number(SCFavg).toFixed(4);
        var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
        var SDDavgString = Number(SDDavg).toFixed(0);

        // Convert SDD day of water year integer to date.
        var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
        var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

        // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
        var newString = ee.String('Region-averaged metrics:')
                          .cat(ee.String('\n\tWater year: '))
                          .cat(ee.String(waterYearDropdown.getValue()))
                          .cat(ee.String('\n\tSCF: '))
                          .cat(ee.String(SCFavgString))
                          .cat(ee.String('\n\tSDD: '))
                          .cat(ee.String(SDDavgString))
                          .cat(ee.String(' ('))
                          .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

        newString.evaluate(function(val) {regionDetails.setValue(val)});
        // Display download button - not implemented in v1.0.
        //downloadButtonPanel.style().set('shown', true);
      }
    }
  });

  // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
  elevationFilterCheckbox = ui.Checkbox({
    value: false,
    label: 'Enable elevation filter',
    onChange: function(checked) {
      elevationFilterPanel.style().set('shown', checked);
    }
  });

  elevationFilterPanel = ui.Panel();
  elevationFilterPanel.add(makeElevationFilter());
  elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

  // Return widgets packaged in a panel.
  geoPanel2.clear();
  //geoDropdown2.setPlaceholder(placeholder);
  return ui.Panel([elevationFilterCheckbox,
                   elevationFilterPanel, submit],
                  ui.Panel.Layout.flow('vertical'));
}



// Geometry textbox for manual entry of HUC, Bounding Box.
// =======================================================
function makeGeoTextbox(key) {
  var geoType = key;
  var instructions;
  // Clear and turn off drawing tools.
  map.drawingTools().clear();
  map.drawingTools().setShown(false);
  // Different constructions for HUC and Bounding Box.
  switch (key) {
    case 'USGS HUC':
      instructions = ui.Label({
        value: 'Enter a 4, 6, 8, or 10 digit USGS HUC. \
                \nMore information can be found using the HUC list:\n' +
                'water.usgs.gov/GIS/huc_name.html',
                //'<a href="water.usgs.gov/GIS/huc_name.html" target="_blank”> Test </a>',

        style: {
          fontWeight: '100',
          padding: '5px',
          whiteSpace: 'pre'
        }
      });
      var hucTextbox = ui.Textbox({
        placeholder: 'Enter HUC',
        onChange: function(text) {
          hucTextbox.setValue(text);
        }
      });

      // Submit button - sets HUC vars based on number of digits, adds data to map.
      var submitButton = ui.Button({
        label: 'Submit',
        onClick: function() {
          var text = hucTextbox.getValue();
          urlGeo = 'HUC' + text;
          var hucs;
          var hucNameProperty;
          switch (text.length) {
            case 4:
              hucs = huc04;
              hucNameProperty = 'huc4';
              break;
            case 6:
              hucs = huc06;
              hucNameProperty = 'huc6';
              break;
            case 8:
              hucs = huc08;
              hucNameProperty = 'huc8';
              break;
            case 10:
              hucs = huc10;
              hucNameProperty = 'huc10';
              break;
            default:
            // Breaking error message for invalid HUCs.
              alert('Sorry, your entry was not a valid HUC. \
              \nPlease enter a 4, 6, 8, or 10 digit HUC.');
          }

          var newGeoList = hucs.filter(ee.Filter.eq(hucNameProperty, text)).toList(1);
          if (newGeoList.size().getInfo() === 0) {
            // Valid number of digits, but entered HUC does not exist in database.
            alert('Sorry, your HUC was not recognized. Please check \
            \nwater.usgs.gov/GIS/huc_name.html and try again.');
          }
          else {
            // Always add selected region to map.
            newGeo = ee.Feature(newGeoList.get(0)).geometry();
            removeMapLayer('Selected region');
            removeMapLayer('SCF');
            removeMapLayer('SDD');
            map.addLayer(newGeo, null, 'Selected region', true, 0.5);
            map.centerObject(newGeo, 6);
            // Only add data layers if a water year is selected.
            var waterYear = waterYearDropdown.getValue();
            if (waterYear > 0) {
              removeMapLayer('SNOTEL sites');
              SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
              SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');
              SDD.updateMask(waterMask.bitwiseNot());
              SCF.updateMask(waterMask.bitwiseNot());
              // Elevation filter
              if (elevationFilterCheckbox.getValue()) {
                var elevMask;
                var srtmRegion = srtm.clip(newGeo);
                var minElev = parseFloat(minElevationTextbox.getValue());
                var maxElev = parseFloat(maxElevationTextbox.getValue());
                // Check for which numbers (min and/or max) were entered
                if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
                  var minElevMask = srtmRegion.select('elevation').gte(minElev);
                  var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
                  elevMask = minElevMask.bitwiseAnd(maxElevMask);
                }
                else if (! isNaN(minElev)) { // number entered only for min elevation
                  elevMask = srtm.select('elevation').gte(minElev);
                }
                else if (! isNaN(maxElev)) { // number only entered for max elevation
                  elevMask = srtm.select('elevation').lte(maxElev);
                }
                // Update SCF and SDD with mask
                SCF = SCF.updateMask(elevMask);
                SDD = SDD.updateMask(elevMask);
              }
              map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
              map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
              map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

              // Calculate region-averaged metrics.
              regionDetailsPanel.style().set('shown', true);
              regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
              var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
              var SCFavgString = Number(SCFavg).toFixed(4);
              var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
              var SDDavgString = Number(SDDavg).toFixed(0);

              // Convert SDD day of water year integer to date.
              var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
              var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

              // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
              var newString = ee.String('Region-averaged metrics:')
                                .cat(ee.String('\n\tWater year: '))
                                .cat(ee.String(waterYearDropdown.getValue()))
                                .cat(ee.String('\n\tSCF: '))
                                .cat(ee.String(SCFavgString))
                                .cat(ee.String('\n\tSDD: '))
                                .cat(ee.String(SDDavgString))
                                .cat(ee.String(' ('))
                                .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

              newString.evaluate(function(val) {regionDetails.setValue(val)});

              // Display download button - not implemented in v1.0.
              //downloadButtonPanel.style().set('shown', true);
            }
          }
        }
      });

      // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
      elevationFilterCheckbox = ui.Checkbox({
        value: false,
        label: 'Enable elevation filter',
        onChange: function(checked) {
          elevationFilterPanel.style().set('shown', checked);
        }
      });
      elevationFilterPanel = ui.Panel();
      elevationFilterPanel.add(makeElevationFilter());
      elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

      // Return widgets packaged in a panel.
      geoPanel2.clear();
      return ui.Panel([instructions, hucTextbox, elevationFilterCheckbox,
                       elevationFilterPanel, submitButton],
                  ui.Panel.Layout.flow('vertical'));

    // Bounding Box option.
    case 'Bounding Box':
      urlGeo = 'bbox';
      instructions = ui.Label({
        value: 'Enter coordinates for lower left/upper right corner of bounding box.',
        style: {
          fontWeight: '100',
          padding: '5px',
        }
      });
      // Lower left corner lat/lon
      var LLlabel = ui.Label({
        value: 'Lower left:',
        style: {stretch: 'vertical', fontWeight: '100'}
      });
      var LLlat = ui.Textbox({
        placeholder: 'LL Latitude',
        onChange: function(text) {
          LLlat.setValue(text);
        }
      });
      var LLlon = ui.Textbox({
        placeholder: 'LL Longitude',
        onChange: function(text) {
          LLlon.setValue(text);
        }
      });
      // Upper right corner lon/lat
      var URlabel = ui.Label({
        value: 'Upper right:',
        style: {fontWeight: '100'}
      });
      var URlat = ui.Textbox({
        style: {width: '136px'},
        placeholder: 'UR Latitude',
        onChange: function(text) {
          URlat.setValue(text);
        }
      });
      var URlon = ui.Textbox({
        placeholder: 'UR Longitude',
        onChange: function(text) {
          URlon.setValue(text);
        }
      });
      // Construct LL/UR panels
      var LL = ui.Panel([LLlabel, LLlat, LLlon],
                        ui.Panel.Layout.flow('horizontal'));
      var UR = ui.Panel([URlabel, URlat, URlon],
                        ui.Panel.Layout.flow('horizontal'));

      // Submit button - checks for valid construction, then adds geometry/data to map.
      submitButton = ui.Button({
        label: 'Submit',
        onClick: function() {
          var LLlatParsed = parseFloat(LLlon.getValue());
          var LLlonParsed = parseFloat(LLlat.getValue());
          var URlatParsed = parseFloat(URlon.getValue());
          var URlonParsed = parseFloat(URlat.getValue());

          // Handle exceptions for bad bbox geometry.
          var goodLon = null;
          var goodLat = null;
          var goodConstruction = null;

          // Longitude check.
          if (LLlonParsed < 0) { //Western hemisphere
            if (URlonParsed < LLlonParsed) {
              alert('Error in bounding box coordinates.\nPlease check lon/lat values and try again.');
            }
            else {
              goodLon = true;
            }
          }
          else { //Eastern hemisphere
            if (LLlonParsed > URlonParsed) {
              alert('Error in bounding box coordinates.\nPlease check lon/lat values and try again.');
            }
            else {
              goodLon = true;
            }
          }
          // Latitude check - both hemispheres have same condition.
          if (URlatParsed < LLlatParsed) {
            alert('Error in bounding box coordinates.\nPlease check lon/lat values and try again.');
          }
          else {
            goodLat = true;
          }
          if (goodLon && goodLat) {
            goodConstruction = true;
          }

          // After confirming good bbox coordinates, add geometry/data to map.
          if (goodConstruction) {
            var LLcnr = ee.Geometry.Point([parseFloat(LLlon.getValue()), parseFloat(LLlat.getValue())]);
            var URcnr = ee.Geometry.Point([parseFloat(URlon.getValue()), parseFloat(URlat.getValue())]);
            newGeo = ee.Geometry.Rectangle([LLcnr, URcnr]);
            // Always add selected region to map.
            removeMapLayer('Selected region');
            removeMapLayer('SCF');
            removeMapLayer('SDD');
            map.addLayer(newGeo, null, 'Selected region', true, 0.5);
            map.centerObject(newGeo, 6);
            // Only add data layers if a water year is selected.
            var waterYear = waterYearDropdown.getValue();
            if (waterYear > 0) {
              removeMapLayer('SNOTEL sites');
              SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
              SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');
              // Elevation filter
              if (elevationFilterCheckbox.getValue()) {
                var elevMask;
                var srtmRegion = srtm.clip(newGeo);
                var minElev = parseFloat(minElevationTextbox.getValue());
                var maxElev = parseFloat(maxElevationTextbox.getValue());
                // Check for which numbers (min and/or max) were entered
                if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
                  var minElevMask = srtmRegion.select('elevation').gte(minElev);
                  var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
                  elevMask = minElevMask.bitwiseAnd(maxElevMask);
                }
                else if (! isNaN(minElev)) { // number entered only for min elevation
                  elevMask = srtm.select('elevation').gte(minElev);
                }
                else if (! isNaN(maxElev)) { // number only entered for max elevation
                  elevMask = srtm.select('elevation').lte(maxElev);
                }
                // Update SCF and SDD with mask
                SCF = SCF.updateMask(elevMask);
                SDD = SDD.updateMask(elevMask);
              }
              map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
              map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
              map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

              // Calculate region-averaged metrics.
              regionDetailsPanel.style().set('shown', true);
              regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
              var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
              var SCFavgString = Number(SCFavg).toFixed(4);
              var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
              var SDDavgString = Number(SDDavg).toFixed(0);

              // Convert SDD day of water year integer to date.
              var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
              var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

              // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
              var newString = ee.String('Region-averaged metrics:')
                                .cat(ee.String('\n\tWater year: '))
                                .cat(ee.String(waterYearDropdown.getValue()))
                                .cat(ee.String('\n\tSCF: '))
                                .cat(ee.String(SCFavgString))
                                .cat(ee.String('\n\tSDD: '))
                                .cat(ee.String(SDDavgString))
                                .cat(ee.String(' ('))
                                .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

              newString.evaluate(function(val) {regionDetails.setValue(val)});

              // Display download button - not implemented in v1.0.
              //downloadButtonPanel.style().set('shown', true);
            }
          }
        }
      });

      // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
      elevationFilterCheckbox = ui.Checkbox({
        value: false,
        label: 'Enable elevation filter',
        onChange: function(checked) {
          elevationFilterPanel.style().set('shown', checked);
        }
      });
      elevationFilterPanel = ui.Panel();
      elevationFilterPanel.add(makeElevationFilter());
      elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

      // Return widgets packaged in a panel.
      geoPanel2.clear();
      return ui.Panel([instructions, LL, UR, elevationFilterCheckbox,
                       elevationFilterPanel, submitButton],
                       ui.Panel.Layout.flow('vertical'));

    // User-uploaded shapefile
    case 'User Asset':
      instructions = ui.Label({
        value: 'Enter the path to your path to your GEE asset, \
                \ne.g. users/yourUsername/path/to/asset \
                \nPlease ensure that your asset is shared publicly (check the box \
                \nlabeled \'Anyone can read\' in the asset sharing options).',
        style: {
          fontWeight: '100',
          padding: '5px',
          whiteSpace: 'pre'
        }
      });
      var assetTextbox = ui.Textbox({
        placeholder: 'Enter asset path',
        style: {width: '250px'},
        onChange: function(text) {
          assetTextbox.setValue(text);
        }
      });

      // Submit button - grabs shapefile, adds data to map.
      submitButton = ui.Button({
        label: 'Submit',
        onClick: function() {
          urlGeo = 'customAsset';
          var newGeoFC;
          var userAsset = assetTextbox.getValue();
          try {
            newGeoFC = ee.FeatureCollection(userAsset);
          }
          catch(err) {
            alert('Sorry, there was a problem with your asset. Please check the path and sharing options and try again.');
          }

          if (typeof newGeoFC !== 'undefined') {
            var newGeoList = newGeoFC.toList(1);
            // Always add selected region to map.
            newGeo = ee.Feature(newGeoList.get(0)).geometry();
            removeMapLayer('Selected region');
            removeMapLayer('SCF');
            removeMapLayer('SDD');
            map.addLayer(newGeo, null, 'Selected region', true, 0.5);
            map.centerObject(newGeo, 6);
            // Only add data layers if a water year is selected.
            var waterYear = waterYearDropdown.getValue();
            if (waterYear > 0) {
              removeMapLayer('SNOTEL sites');
              SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
              SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');
              SDD.updateMask(waterMask.bitwiseNot());
              SCF.updateMask(waterMask.bitwiseNot());
              // Elevation filter
              if (elevationFilterCheckbox.getValue()) {
                var elevMask;
                var srtmRegion = srtm.clip(newGeo);
                var minElev = parseFloat(minElevationTextbox.getValue());
                var maxElev = parseFloat(maxElevationTextbox.getValue());
                // Check for which numbers (min and/or max) were entered
                if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
                  var minElevMask = srtmRegion.select('elevation').gte(minElev);
                  var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
                  elevMask = minElevMask.bitwiseAnd(maxElevMask);
                }
                else if (! isNaN(minElev)) { // number entered only for min elevation
                  elevMask = srtm.select('elevation').gte(minElev);
                }
                else if (! isNaN(maxElev)) { // number only entered for max elevation
                  elevMask = srtm.select('elevation').lte(maxElev);
                }
                // Update SCF and SDD with mask
                SCF = SCF.updateMask(elevMask);
                SDD = SDD.updateMask(elevMask);
              }
              map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
              map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
              map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

              regionDetailsPanel.style().set('shown', true);
              regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
              var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
              var SCFavgString = Number(SCFavg).toFixed(4);
              var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
              var SDDavgString = Number(SDDavg).toFixed(0);

              // Convert SDD day of water year integer to date.
              var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
              var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

              // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
              var newString = ee.String('Region-averaged metrics:')
                                .cat(ee.String('\n\tWater year: '))
                                .cat(ee.String(waterYearDropdown.getValue()))
                                .cat(ee.String('\n\tSCF: '))
                                .cat(ee.String(SCFavgString))
                                .cat(ee.String('\n\tSDD: '))
                                .cat(ee.String(SDDavgString))
                                .cat(ee.String(' ('))
                                .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

              newString.evaluate(function(val) {regionDetails.setValue(val)});

              // Display download button - not implemented in v1.0.
              //downloadButtonPanel.style().set('shown', true);
            }
          }
        }
      });


        // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
        elevationFilterCheckbox = ui.Checkbox({
          value: false,
          label: 'Enable elevation filter',
          onChange: function(checked) {
            elevationFilterPanel.style().set('shown', checked);
          }
        });
        elevationFilterPanel = ui.Panel();
        elevationFilterPanel.add(makeElevationFilter());
        elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

        // Return widgets packaged in a panel.
        geoPanel2.clear();
        return ui.Panel([instructions, assetTextbox, elevationFilterCheckbox,
                         elevationFilterPanel, submitButton],
                    ui.Panel.Layout.flow('vertical'));

  }
}


// Set up drawing tools for a user-drawn polygon or rectangle.
// ===========================================================
function userDrawnPolygon() {
  // Turn on drawing tools.
  map.drawingTools().setShown(true);
  map.drawingTools().clear();
  map.drawingTools().addLayer({
    geometries: [],
    name: 'Selected region',
    color: 'black'
  });

  var drawingInstructions = ui.Label({
    value: 'Use the drawing tools in the upper left corner \nof the map to draw your own polygon or rectangle. \nAfterwards, filter elevation if desired, then click Submit.',
    style: {
      fontWeight: '100',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });

  // Submit button - adds geometry and data layers to map.
  var submit = ui.Button({
    label: 'Submit',
    onClick: function() {
      // Convert drawn shape to newGeo.
      newGeo = map.drawingTools().layers().get(0).toGeometry();
      removeMapLayer('SCF');
      removeMapLayer('SDD');
      // Only add data layers if a water year is selected.
      var waterYear = waterYearDropdown.getValue();
      if (waterYear > 0) {
        // Turn off the layer w/ the drawn polygon to see data better.
        map.drawingTools().layers().get(0).setShown(false);
        removeMapLayer('SNOTEL sites');
        SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
        SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');
        // Elevation filter
        if (elevationFilterCheckbox.getValue()) {
          var elevMask;
          var srtmRegion = srtm.clip(newGeo);
          var minElev = parseFloat(minElevationTextbox.getValue());
          var maxElev = parseFloat(maxElevationTextbox.getValue());
          // Check for which numbers (min and/or max) were entered
          if (! isNaN(minElev) && ! isNaN(maxElev)) { // numbers entered for both filters
            var minElevMask = srtmRegion.select('elevation').gte(minElev);
            var maxElevMask = srtmRegion.select('elevation').lte(maxElev);
            elevMask = minElevMask.bitwiseAnd(maxElevMask);
          }
          else if (! isNaN(minElev)) { // number entered only for min elevation
            elevMask = srtm.select('elevation').gte(minElev);
          }
          else if (! isNaN(maxElev)) { // number only entered for max elevation
            elevMask = srtm.select('elevation').lte(maxElev);
          }
          // Update SCF and SDD with mask
          SCF = SCF.updateMask(elevMask);
          SDD = SDD.updateMask(elevMask);
        }
        map.addLayer(SDD.clip(newGeo).updateMask(waterMask.not()), {palette:palette_sdd, min:0, max:365}, 'SDD');
        map.addLayer(SCF.clip(newGeo).updateMask(waterMask.not()), {palette: palette_scf}, 'SCF');
        map.layers().set(map.layers().length(),
                         ui.Map.Layer(snotel.draw({pointRadius: 5, color: 'dd1cff'}),
                                      null, 'SNOTEL sites', snotelCheckbox.getValue()));

        // Calculate region-averaged metrics.
        regionDetailsPanel.style().set('shown', true);
        regionDetails.setValue('Region-averaged metrics:\n\tWater year: Loading...\n\tSCF: Loading...\n\tSDD: Loading...');
        var SCFavg = SCF.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('ccSCF').getInfo();
        var SCFavgString = Number(SCFavg).toFixed(4);
        var SDDavg = SDD.reduceRegion({
                           reducer: ee.Reducer.mean(),
                           geometry: newGeo,
                           bestEffort: true}).get('SDD').getInfo();
        var SDDavgString = Number(SDDavg).toFixed(0);

        // Convert SDD day of water year integer to date.
        var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
        var SDDdisplay = startDate.advance(SDDavg, 'day').format('MMMM dd, YYYY');

        // Asynchronous update of region-averaged metrics (using evaluate, not getInfo)
        var newString = ee.String('Region-averaged metrics:')
                          .cat(ee.String('\n\tWater year: '))
                          .cat(ee.String(waterYearDropdown.getValue()))
                          .cat(ee.String('\n\tSCF: '))
                          .cat(ee.String(SCFavgString))
                          .cat(ee.String('\n\tSDD: '))
                          .cat(ee.String(SDDavgString))
                          .cat(ee.String(' ('))
                          .cat(ee.String(SDDdisplay)).cat(ee.String(')'));

        newString.evaluate(function(val) {regionDetails.setValue(val)});

        // Display download button - not implemented in v1.0.
        //downloadButtonPanel.style().set('shown', true);
      }
    }
  });

  var resetDrawingButton = ui.Button({
    label: 'Reset drawing',
    onClick: function() {
      removeMapLayer('SCF');
      removeMapLayer('SDD');
      map.drawingTools().clear();
      map.drawingTools().addLayer({
        geometries: [],
        name: 'Selected region',
        color: 'gray'
      });
    }
  });

  // Create filter checkbox/panel in this scope to avoid breaking app upon geometry change.
  elevationFilterCheckbox = ui.Checkbox({
    value: false,
    label: 'Enable elevation filter',
    onChange: function(checked) {
      elevationFilterPanel.style().set('shown', checked);
    }
  });
  elevationFilterPanel = ui.Panel();
  elevationFilterPanel.add(makeElevationFilter());
  elevationFilterPanel.style().set('shown', elevationFilterCheckbox.getValue());

  // Return widgets packaged in a panel.
  geoPanel2.clear();
  var buttons = ui.Panel([submit, resetDrawingButton],
                          ui.Panel.Layout.flow('horizontal'));
  return ui.Panel([drawingInstructions, elevationFilterCheckbox,
                   elevationFilterPanel, buttons],
                   ui.Panel.Layout.flow('vertical'));
}


// Generate download URL for image containing SCF/SDD data clipped to region.
// ==========================================================================
var downloadButton = ui.Button({
  label: 'Generate URL to download data for this region',
  style: {whiteSpace: 'pre'},
  onClick: function() {
    var dataIC = ee.ImageCollection([SCF, SDD]);
    var dataImage = dataIC.toBands().rename(['SCF','SDD']);
    var urlName = 'SCM_WY' + waterYearDropdown.getValue() + '_' + urlGeo;
    try {
      var url = dataImage.getDownloadURL({name:urlName, region:newGeo});
      alert('Copy/paste the following URL into a new tab to download compressed GeoTIFF files containing SCF/SDD data for your selected region:\n\n(Click OK to continue)');
      alert(url);
    }
    catch(err) {
      alert('Sorry, your selected region is too large to download.');
    }
  }
});


////////////////////////////////////////////////////////
//      Clicked point data and timeseries charts      //
////////////////////////////////////////////////////////

// Initialize label to hold clicked point SCF/SDD details.
// =======================================================
var pointDetails = ui.Label({
  value: '',
  style: {
    fontSize: '14px',
    fontWeight: '100',
    padding: '5px',
    whiteSpace: 'pre'
  }
});


// Show point on the map.
// ======================
function showPointOnMap(point) {
  if (pointColorCounter >= 7) {
    alert('A maximum of 8 points can be added to the map.\nPlease reset the points to select new locations.');
  }
  else {
    pointColor = pointColors[pointColorCounter];
    pointColorDisplay = pointColor.charAt(0).toUpperCase() + pointColor.slice(1);
    // Update label before asynchronous evaluation.
    pointDetails.setValue('Coordinates: Loading...\nDisplay color: Loading...\
                           \nWater year: Loading...\
                           \nSCF: Loading...\nSDD: Loading...');
    // Create new geometry, add to map as top-most layer for visibility
    var dot = ui.Map.Layer(point, {color: pointColor}, 'Point ' + pointNameCounter.toString());
    map.layers().set(map.layers().length(), dot);
    // Increment counters for color and layer name
    pointColorCounter++;
    pointNameCounter++;
  }
}


// Get and display point details - coordinates, WY, SCF, SDD.
// ==========================================================
var timeseriesCoords; //need global scope to pass to chart
function getPointDetails(point) {
  if (waterYear < 10) { //no water year selection
    alert('Please select a water year to display SCF and SDD data at your selected point.');
  }
  else { //water year has been selected
    pointDetailsPanel.style().set('shown', true);
    showPointOnMap(point);
    // Some regex matches to cut down on displayed decimals in panel
    timeseriesCoords = ee.Geometry(point).coordinates();
    var lonString = String(timeseriesCoords.get(0).getInfo()).match(/.+\.\d{5}/)[0];
    var latString = String(timeseriesCoords.get(1).getInfo()).match(/.+\.\d{5}/)[0];
    var waterYear = waterYearDropdown.getValue();
    // Reduce SCF/SDD just at the selected point.
    var SCF = ee.Image('users/ryanlcrumley/metrics/wyr/SCF'+waterYear+'global_15');
    var SDD = ee.Image('users/ryanlcrumley/metrics/wyr/SDD'+waterYear+'global_15');
    var SCFpoint = SCF.reduceRegion(ee.Reducer.first(), point);
    var SDDpoint = SDD.reduceRegion(ee.Reducer.first(), point);

    // Check for valid data at selected point.
    switch (SCFpoint.get('ccSCF').getInfo()) {
      case null:
        var SCFdataString = 'No data';
        break;
      default:
        var SCFdata = SCFpoint.get('ccSCF').getInfo();
        SCFdataString = Number(SCFdata).toFixed(4);
    }

    switch (SDDpoint.get('SDD').getInfo()) {
      case null:
        var SDDdataString = 'No data';
        break;
      default:
        var SDDdata = SDDpoint.get('SDD');
        var startDate = ee.Date.fromYMD(parseInt(waterYear, 10)-1, 10, 1);
        var SDDdisplay = startDate.advance(SDDdata, 'day').format('MMMM dd, YYYY');
        SDDdataString = ee.String(SDDdata)
                          .cat(ee.String(' ('))
                          .cat(ee.String(SDDdisplay)).cat(ee.String(')'));
    }

    // Asynchronous update of metrics at selected point (using evaluate, not getInfo)
    var newString = ee.String('Coordinates: ')
                      .cat(ee.String(lonString)).cat(ee.String(', '))
                      .cat(ee.String(latString))
                      .cat(ee.String('\nDisplay color: '))
                      .cat(ee.String(pointColorDisplay))
                      .cat(ee.String('\nWater year: '))
                      .cat(ee.String(waterYear))
                      .cat(ee.String('\nSCF: '))
                      .cat(ee.String(SCFdataString))
                      .cat(ee.String('\nSDD: '))
                      .cat(ee.String(SDDdataString));

    newString.evaluate(function(val) {pointDetails.setValue(val)});
  }
}


// Timeseries charts for SCF/SDD for all water years at all selected points.
// =========================================================================
function makeTimeseriesButtons() {
  var pointToTimeseriesButton = ui.Button({
    label: 'Add point(s) to timeseries charts',
    onClick: function() {
      chartPanel.clear();
      var points = collectPointsOnMap();
      // SCF timeseries
      var SCFchart = ui.Chart.image.seriesByRegion({
        imageCollection: SCF_IC,
        regions: points,
        reducer: ee.Reducer.first(),
        xProperty: 'system:time_start',
        seriesProperty: 'label'
      })
      .setChartType('ScatterChart')
      .setOptions({
        title: '',
        vAxis: {title: 'Snow Cover Frequency'},
        haxis: {title: 'Water Year'},
        lineWidth: 1,
        pointSize: 4,
        series: {
          0: {color: pointColors[0]},
          1: {color: pointColors[1]},
          2: {color: pointColors[2]},
          3: {color: pointColors[3]},
          4: {color: pointColors[4]},
          5: {color: pointColors[5]},
          6: {color: pointColors[6]},
          7: {color: pointColors[7]}
      }});

      //SDD timeseries
      var SDDchart = ui.Chart.image.seriesByRegion({
        imageCollection: SDD_IC,
        regions: points,
        reducer: ee.Reducer.first(),
        xProperty: 'system:time_start',
        seriesProperty: 'label'
      })
      .setChartType('LineChart')
      .setOptions({
        title: '',
        vAxis: {title: 'Snow Disappearance Date [DoWY]'},
        haxis: {title: 'Water Year'},
        lineWidth: 1,
        pointSize: 4,
        series: {
          0: {color: pointColors[0]},
          1: {color: pointColors[1]},
          2: {color: pointColors[2]},
          3: {color: pointColors[3]},
          4: {color: pointColors[4]},
          5: {color: pointColors[5]},
          6: {color: pointColors[6]},
          7: {color: pointColors[7]}
      }});

      var charts = ui.Panel([SCFchart, SDDchart], ui.Panel.Layout.flow('vertical'));
      chartPanel.add(charts);
    }
  });

  // Reset button for charts.
  var resetTimeseriesButton = ui.Button({
    label: 'Reset points and charts',
    onClick: function(){
      chartPanel.clear();
      pointDetailsPanel.style().set('shown', false);
      removePointsFromMap();
      pointColorCounter = 0;
      pointNameCounter = 1;
    }
  });

  // Return buttons packaged in panel.
  return ui.Panel([pointToTimeseriesButton, resetTimeseriesButton],
                  ui.Panel.Layout.flow('horizontal'));
}


/////////////////////////////////////////////////////
/////////////////////////////////////////////////////
//      Initialize and create root map/panels      //
/////////////////////////////////////////////////////
/////////////////////////////////////////////////////

// Initialize the widgets.
// =======================
function init() {
  var panel = ui.Panel({
    style: {
      width: '30%',
      border: '3px solid #ddd',
    }
  });
  var title = ui.Label({
    value: 'SnowCloudMetrics app',
    style: {
      fontSize: '18px',
      fontWeight: '100',
      padding: '5px',
    }
  });
  var description = ui.Label({
    value: '\nPlease scroll down in this side panel to see \
            \nall available options.',
    style: {
      color: 'black',
      fontSize: '16px',
      fontWeight: '100',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });
  var requiredParams = ui.Label({
    value: '_________________________________________\
          \nRequired parameters',
    style: {
      color: 'black',
      fontSize: '16px',
      fontWeight: 'bold',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });
  var waterYearInstructions = ui.Label({
    value: 'Select a water year for SCF/SDD data.',
    style: {
      color: 'black',
      fontWeight: '100',
      padding: '5px',
    }
  });
  var geometryInstructions = ui.Label({
    value: 'Select the region of interest.\
            \nNote that larger regions take longer to process.',
    style: {
      color: 'black',
      fontWeight: '100',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });
  var visualizationOptions = ui.Label({
    value: '_________________________________________\
          \nVisualization options',
    style: {
      color: 'black',
      fontSize: '16px',
      fontWeight: 'bold',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });
  var clickInstructions = ui.Label({
    value: '______________________________________________\
          \nClick the map to show the SCF/SDD values \
          \nfor that pixel.',
    style: {
      color: 'black',
      fontWeight: 'bold',
      padding: '5px',
      whiteSpace: 'pre'
    }
  });

  var resetPanel = ui.Panel();
  var waterYearDropdownPanel = ui.Panel();
  var geoDropdownPanel = ui.Panel();
  var geoPanel2 = ui.Panel();
  var regionDetailsPanel = ui.Panel();
  var downloadButtonPanel = ui.Panel();
  var pointDetailsPanel = ui.Panel();
  var chartPanel = ui.Panel();

  // First stuff - title, reset button, etc
  panel.add(resetPanel);
  panel.add(description);
  // Required parameters
  panel.add(requiredParams);
  panel.add(waterYearInstructions);
  panel.add(waterYearDropdownPanel);
  panel.add(geometryInstructions);
  panel.add(geoDropdownPanel);
  panel.add(geoPanel2);
  panel.add(regionDetailsPanel);
  // Visualization options
  panel.add(visualizationOptions);
  panel.add(snotelCheckbox);
  panel.add(makeColorbarCheckboxes());

  panel.add(downloadButtonPanel);
  panel.add(clickInstructions);
  panel.add(pointDetailsPanel);
  //panel.add(chartPanel);


  var splitPanel = ui.SplitPanel({
    firstPanel: panel,
    secondPanel: map,
  });

  ui.root.clear();
  ui.root.add(splitPanel);
  resetPanel.add(makeTitleAndReset());
  waterYearDropdownPanel.add(makeWaterYearDropdown());
  geoDropdownPanel.add(makeGeometryDropdown());

  regionDetailsPanel.add(regionDetails);
  regionDetailsPanel.style().set('shown', false);

  downloadButtonPanel.add(downloadButton);
  downloadButtonPanel.style().set('shown', false);

  pointDetailsPanel.add(pointDetails);
  pointDetailsPanel.add(chartPanel);
  pointDetailsPanel.add(makeTimeseriesButtons());
  pointDetailsPanel.style().set('shown', false);
  chartPanel.clear();

  // Bind the click handler to the new map.
  map.onClick(function(coordinates){
    if (waterYearDropdown.getValue() < 10) {
      alert('Please select a water year to display SCF and SDD data at your selected point.');
    }
    else {
      var point = ee.Geometry.Point([coordinates.lon, coordinates.lat]);
      getPointDetails(point);
    }
  });

return [waterYearDropdownPanel, geoDropdownPanel, geoPanel2,
        regionDetailsPanel, downloadButtonPanel, pointDetailsPanel, chartPanel];
}

var panels = init();
waterYearDropdownPanel = panels[0];
geoDropdownPanel = panels[1];
geoPanel2 = panels[2];
regionDetailsPanel = panels[3];
downloadButtonPanel = panels[4];
pointDetailsPanel = panels[5];
chartPanel = panels[6];
