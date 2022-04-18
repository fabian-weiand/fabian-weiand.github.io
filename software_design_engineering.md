## Project 1

# Software Design and Engineering

This artifact is a full stack application for an animal shelter using Jupyter, Python, and MongoDB which was built in Client Server Development class during program. 
It showcases my skills both for front and back-end development as well as how to connect the two using a python module. 


```
from jupyter_plotly_dash import JupyterDash

import dash
import dash_leaflet as dl
import dash_core_components as dcc
import dash_html_components as html
import plotly.express as px
import dash_table as dt
from dash.dependencies import Input, Output, State

import base64
import os
import numpy as np
import pandas as pd
from pymongo import MongoClient
from bson.json_util import dumps
from animal_shelter import AnimalShelter

###########################
# Data Manipulation / Model
###########################
username = "aacuser"
password = "aacuser"

shelter = AnimalShelter(username, password, "localhost", 27017, "AAC")
df = pd.DataFrame.from_records(shelter.read(projection={'_id': False}))

#########################
# Dashboard Layout / View
#########################
app = JupyterDash('SimpleExample')

image_filename = 'Grazioso Salvare Logo.png'
encoded_image = base64.b64encode(open(image_filename, 'rb').read())

html.Img(src='data:image/png;base64,{}'.format(encoded_image.decode()))

app.layout = html.Div([
    html.Div(id='hidden-div', style={'display':'none'}),
    html.Hr(),
    html.Center(html.B(children=[html.H1('Grazioso Salvare Animal Shelter Dashboard'),
               html.Img(id='customer-image', height="50", 
                        src='data:image/png;base64,{}'.format(encoded_image.decode()),
                        alt='customer image')])),
    html.Center(html.B(html.H2('Developed by Fabian Weiand'))),
    html.Hr(),
    html.Div(className='row',
             style={"width": "50%"},
             children=[
                 # Water Rescue
                 # Mountain or Wilderness Rescue
                 # Disaster Rescue or Individual Tracking
                 # Reset (returns all widgets to their original, unfiltered state)
                 dcc.Dropdown(
                     id='filter-type',
                     value=None,
                     options=[
                         {"label": "Water Rescue", "value": "Water Rescue"},
                         {"label": "Mountain or Wilderness Rescue", "value": "Mountain or Wilderness Rescue"},
                         {"label": "Disaster Rescue or Individual Tracking", "value": "Disaster Rescue or Individual Tracking"},
                         {"label": "Reset", "value": "Reset"},
                     ]
                 ),

             ]),
    html.Hr(),
    dt.DataTable(
        id='datatable-id',
        style_header={
            'whiteSpace': 'normal',
            'height': 'auto',
        },
        columns=[
            {"name": i, "id": i, "deletable": False, "selectable": True} for i in df.columns
        ],
        data=df.to_dict('records'),
        
        editable=False,
        filter_action='native',
        sort_action='native',
        sort_mode='multi',
        column_selectable=False,
        row_selectable=False,
        row_deletable=False,
        selected_columns=[],
        selected_rows=[],
        page_action='native',
        page_current=0,
        page_size=10,    
    ),
    html.Br(),
    html.Hr(),
#This sets up the dashboard so that your chart and your geolocation chart are side-by-side
    html.Div(className='row',
         style={'display' : 'flex'},
             children=[
        html.Div(
            id='graph-id',
            className='col s12 m6',

            ),
        html.Div(
            id='map-id',
            className='col s12 m6',
            )
        ])
])

#############################################
# Interaction Between Components / Controller
#############################################
    
@app.callback([Output('datatable-id','data'),
               Output('datatable-id','columns')],
              [Input('filter-type', 'value')])
def update_dashboard(filter_type):
    # Water Rescue
    # Mountain or Wilderness Rescue
    # Disaster Rescue or Individual Tracking
    # Reset (returns all widgets to their original, unfiltered state)
    if filter_type == "Water Rescue":
        # Labrador Retriever Mix, Chesapeake Bay Retriever, Newfoundland
        # Intact Female
        # 26 weeks to 156 weeks
        dff=pd.DataFrame.from_records(shelter.read({
                        "breed": {"$in": ["Labrador Retriever Mix", "Chesapeake Bay Retriever", "Newfoundland"]},
                        "sex_upon_outcome": "Intact Female",
                        "age_upon_outcome_in_weeks": {"$gte": 26, "$lte": 156}},
                     projection={'_id': False}))
        data = dff.to_dict('records')
        columns=[{"name": i, "id": i, "deletable": False, "selectable": True} for i in dff.columns]
    elif filter_type == "Mountain or Wilderness Rescue":
        # German Shepherd, Alaskan Malamute, Old English Sheepdog, Siberian Husky, Rottweiler
        # Intact Male
        # 26 weeks to 156 weeks
        dff=pd.DataFrame.from_records(shelter.read({
                        "breed": {"$in": ["German Shepherd", "Alaskan Malamute", "Old English Sheepdog", "Siberian Husky", "Rottweiler"]},
                        "sex_upon_outcome": "Intact Male",
                        "age_upon_outcome_in_weeks": {"$gte": 26, "$lte": 156}},
                     projection={'_id': False}))
        data = dff.to_dict('records')
        columns=[{"name": i, "id": i, "deletable": False, "selectable": True} for i in dff.columns]
    elif filter_type == "Disaster Rescue or Individual Tracking":
        # Doberman Pinscher, German Shepherd, Golden Retriever, Bloodhound, Rottweiler
        # Intact Male
        # 20 weeks to 300 weeks
        dff=pd.DataFrame.from_records(shelter.read({
                        "breed": {"$in": ["Doberman Pinscher", "German Shepherd", "Golden Retriever", "Bloodhound", "Rottweiler"]},
                        "sex_upon_outcome": "Intact Male",
                        "age_upon_outcome_in_weeks": {"$gte": 20, "$lte": 300}},
                     projection={'_id': False}))
        data = dff.to_dict('records')
        columns=[{"name": i, "id": i, "deletable": False, "selectable": True} for i in dff.columns]
    else:
        columns=[{"name": i, "id": i, "deletable": False, "selectable": True} for i in df.columns]
        data=df.to_dict('records')

    return (data, columns)


@app.callback(
    Output('datatable-id', 'style_data_conditional'),
    [Input('datatable-id', 'selected_columns')]
)
def update_styles(selected_columns):
    return [{
        'if': { 'column_id': i },
        'background_color': '#D2F3FF'
    } for i in selected_columns]


@app.callback(
    Output('graph-id', "children"),
    [Input('datatable-id', "derived_viewport_data")])
def update_graphs(viewData):
    # retrive the pandas dataFrame for the view data
    dff = pd.DataFrame.from_dict(viewData)
    return [
       dcc.Graph(
           figure = px.pie(dff["breed"], names=dff["breed"])
       )
    ]


@app.callback(
    Output('map-id', "children"),
    [Input('datatable-id', "derived_viewport_data")])
def update_map(viewData):

    # retrive the pandas dataFrame for the view data
    dff = pd.DataFrame.from_dict(viewData)

    # initialize the marker list, defining the tile layer
    markers = [dl.TileLayer(id="base-layer-id")]

    for index in dff.index:
        # extend is more efficient than concatenation +=
        markers.extend([dl.Marker(position=[dff["location_lat"][index], dff["location_long"][index]],
                              children=[
                                  dl.Tooltip(dff["name"][index] if dff["name"][index] else "Not Named"),
                                  dl.Popup([
                                      html.H2(f"Name: {dff['name'][index] if dff['name'][index] else 'Not Named'}"),
                                      html.P(f"Type: {dff['animal_type'][index]}"),
                                      html.P(f"Breed: {dff['breed'][index]}"),
                                      html.P(f"Color: {dff['color'][index]}"),
                                      html.P(f"Sex: {dff['sex_upon_outcome'][index]}")
                                  ])
                              ]
                              )
                    ])
    return [
        dl.Map(style={'width': '1000px', 'height': '500px'},
               # center the geo map so that all points are visible
               center=[
                   (np.max(dff['location_lat'].to_numpy()) + np.min(dff['location_lat'].to_numpy())) / 2.0,
                   (np.max(dff['location_long'].to_numpy()) + np.min(dff['location_long'].to_numpy())) / 2.0
               ],
               zoom=10,
               # if wanting to show just the first Dog entry use children=markers[:2]
               # if wanting to show ALL Dogs in the current viewable table use children=markers
               children=markers
               )
    ]


app
```

```
from pymongo import MongoClient
from bson.objectid import ObjectId


class AnimalShelter(object):
    """ CRUD operations for Animal collection in MongoDB """

    def __init__(self, username, password, host, port, database):
        # Initializing the MongoClient. This helps to
        # access the MongoDB databases and collections.
        self.client = MongoClient(
            f'mongodb://{username}:{password}@{host}:{port}/{database}')
        self.database = self.client[f'{database}']

        # default collection is animals. Caller can change this with set_collection(str) method
        self.collection = None
        self.set_collection()

    def set_collection(self, collection="animals"):
        """
            The set_collection method is a setter method for changing the database collection.
            By default this is set to "animals".
            :param collection:
            :return:
        """
        self.collection = self.database[collection]

    def create(self, data=None) -> bool:
        """
            The create method inserts a document into the database collection
            :param data: dict type object to be inserted to the database
            :return: bool for success or failure
        """
        if data is not None:
            # insert the document to the database. insert() is deprecated, using insert_one()
            result = self.collection.insert_one(
                data)  # data should be dictionary
            # retrieve the document that was just inserted
            result = self.collection.find_one(data)

            # validate the document was inserted
            if result is not None:
                return True
        else:
            raise Exception("Nothing to save, because data parameter is empty")

        return False

    def read(self, query_filter=None, **kwargs) -> dict:
        """
            The read method will find a document in the database and return it to the user
            :param query_filter: (Optional) dict object to filter the find command. If left blank then all documents are
            retrieved.
            :param kwargs: see a list of arguments that can be passed to the find() function
            https://pymongo.readthedocs.io/en/stable/api/pymongo/collection.html#pymongo.collection.Collection.find
            :return: list of dict objects are returned
        """
        # check if parameter is None and set it to a default
        if query_filter is None:
            query_filter = {}

        return self.collection.find(filter=query_filter, **kwargs)

    def update(self, update_filter=None, update=None, upsert=False) -> bool:
        """
            The update method will accespt an update filter and an update set to send to the database
            :param update_filter: a query that matches the document to update.
            :param update: The modifications to apply to the document.
            :param upsert: If True, perform an insert if no documents match the filter.
            :return: bool True if modified count is 1.
        """

        if not isinstance(update, dict) or not isinstance(update_filter, dict):
            raise Exception(f"invalid parameter type:\nupdate_filter: {type(update_filter)}\nupdate: {type(update)}\n"
                            f"Expected dict, dict")

        result = self.collection.update_one(
            filter=update_filter, update=update, upsert=upsert)
        if result.modified_count == 1:
            return True

        return False

    def delete(self, delete_filter=None) -> bool:
        """
            The delete method will take a single document and delete it from the database
            :param delete_filter: rdict type object that is required in order to delete the document from the database
            :return: returns a bool for success or failure
        """

        # validate the query is not None
        if not isinstance(delete_filter, dict):
            print(type(delete_filter))
            raise Exception("invalid parameter: delete_filter")

        result = self.collection.delete_one(delete_filter)
        # result is a DeletedResult object with a count of deleted items
        # if one item is deleted then return true
        if result.deleted_count == 1:
            return True

        return False
```
