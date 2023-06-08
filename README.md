# Set-Type-Parameter
import clr
from Autodesk.Revit.DB import *
from Autodesk.Revit.DB import BuiltInParameter
from Autodesk.Revit.UI.Selection import ObjectType
from Autodesk.Revit.DB import Transaction
import rpw
from rpw.ui.forms import (Console, FlexForm, Label, TextBox, Button)
clr.AddReference('RevitAPI')
clr.AddReference('RevitAPIUI')
clr.AddReference("RevitNodes")
doc = __revit__.ActiveUIDocument.Document
uidoc = __revit__.ActiveUIDocument
from pyrevit import UI
from pyrevit import script
clr.AddReference('System.Windows.Forms')
clr.AddReference('IronPython.Wpf')
from Autodesk.Revit.DB import FilteredElementCollector, BuiltInCategory, Transaction
from rpw import revit, uidoc, ui, db

def get_type_by_name(type_name):
    """Function to get Type by Name."""
    # CREATE RULE
    param_id = ElementId(BuiltInParameter.ALL_MODEL_TYPE_NAME)
    f_param = ParameterValueProvider(param_id)
    f_evaluator = FilterStringEquals()
    f_rule = FilterStringRule(f_param, f_evaluator, type_name, True)
    # CREATE FILTER
    filter_type_name = ElementParameterFilter(f_rule)
    # GET ELEMENTS
    return FilteredElementCollector(doc).WherePasses(filter_type_name) \
        .WhereElementIsElementType().FirstElement()
type_name = "45x76"
collector = FilteredElementCollector(doc)
elementTypes = collector.OfClass(ElementType).ToElements()
elementType = next((e for e in elementTypes if e.FamilyName == type_name), None)

if elementType is not None:
    elementTypeId = elementType.Id
else:
    elementTypeId = ElementId.InvalidElementId

    
element_id = ElementId(333856)
element = doc.GetElement(element_id)

parameter_name = "Height"
parameter = element.LookupParameter(parameter_name)




    # Get user input

    args
    type_name = values['typeName']
    param_name = values['param_name']
    _value = float(values['_value'])
    win_num = float(values['textbox'])
    win_num2 = win_num - 1
    win_step = 0.6 / win_num2

    if not type_name:
        print("Please enter a value for 'Element's Type Name'")
        return

    t = Transaction(doc, "Place Windows on Walls")
    t.Start()

    
    element_types = collector.OfClass(ElementType).WhereElementIsElementType().ToElements()

    element_type = next((element_type for element_type in element_types if element_type.Name == type_name), None)

    parameter = element_type.LookupParameter(parameter_name)
    # Find the wall
    w = uidoc.Selection.PickObject(Autodesk.Revit.UI.Selection.ObjectType.Element)
    wall_id = w.ElementId
    wall = doc.GetElement(wall_id)

    if wall is None:
        print("Wall not selected")
    else:
        ref = HostObjectUtils.GetSideFaces(wall, ShellLayerType.Exterior)
        face = wall.GetGeometryObjectFromReference(ref[0])
        edges = face.EdgeLoops
        edge = edges[0]
        vec = XYZ(0, 5, 0)
        hrz_offset = edge[3].AsCurve().CreateOffset(-5, vec)
        r = range_float(0.1, 0.9, win_step)

        for i in r:
            pts = hrz_offset.Evaluate(i, True)
            location = XYZ(pts.X, pts.Y, pts.Z)
            instance = doc.Create.NewFamilyInstance(location, symbol, wall, StructuralType.NonStructural)
            parameter = instance.LookupParameter(param_name)

            if parameter and not parameter.IsReadOnly:
                if parameter.StorageType == StorageType.String:
                    parameter.Set(str(_value))
                elif parameter.StorageType == StorageType.Integer:
                    parameter.Set(int(_value))
                elif parameter.StorageType == StorageType.Double:
                    parameter.Set(_value)
                with Transaction(doc, "Change Parameter") as transaction:
                    transaction.Start()
                    transaction.Commit()
    t.Commit()

def range_float(start, end, step):
    x = start
    while x <= end:
        yield x
        x = x + step

def button_clicked(sender, args):
    bu1_click(sender, args)

# Prompt user to select the wall
wall_selection = uidoc.Selection.PickObject(ObjectType.Element)
wall_id = wall_selection.ElementId
wall = doc.GetElement(wall_id)

if wall is None:
    print("Wall not selected")
else:
    components2 = [
        Label('How many windows to place:'),
        TextBox('textbox'),
        Label("Element's Type Name:"),
        TextBox('type_name', Text="45x76"),
        Label("Family's Type Name:"),
        TextBox('family_name', Text="CAP_FEN"),
        Label('Parameter Name:'),
        TextBox('param_name', Text="Height"),
        Label('Value:'),
        TextBox('_value', Text="2"),
        Button('Select', on_click=button_clicked)
    ]

    form = FlexForm("Placing Window", components2)
    form.show()
def bu1_click(sender, form):
