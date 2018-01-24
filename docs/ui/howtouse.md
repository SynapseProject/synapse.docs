# How To Use

## Plan Execution

### Search And Select A Plan

<p>
<img alt="Search Plan" src="../../img/ui_plan_execution_search_plan.png" />
</p>

To narrow down the number of plans listed in the plan list: 

1. Click to choose between regex "{..}" or in-string "*" filter.
2. Enter the filter expression.  
3. Filtering is done automatically for you as you type the filter expression. You click on this button only if you wish to force a reload. 
4. Click on the desired plan name to select.

### Execute A Plan

<p>
<img alt="Execute Plan" src="../../img/ui_plan_execution_execute_plan.png" />
</p>

1. Follow the steps in ["Search and Select A Plan" section](#search-and-select-a-plan) to select a plan.
2. Click the "Execute" tab if it is not the current tab.
3. Enter the request number (optional)
4. Enter dynamic parameter values (applicable if the plan has dynamic parameters defined). 
5. Check the "Pass blank value" checkbox if you wish to send blank values.
6. Click "Show Diagram" to get a visual representation of the plan (optional).
7. Click the "Execute" button to execute the plan. If you see a "Reset" button instead, click it to restore the "Execute" button.

    The "Reset" button helps to make sure you do not accidentally execute the same plan more than once.
    
8. You should see a notification message like the one below at the bottom left corner of the window. Take note of the "Instance Id" as you will need it to check the status of the execution later on.

    <p>
    <img alt="Plan execution message" src="../../img/ui_plan_execution_execute_plan_msg.png" />
    </p>

Note: Action names with "G" suffix are action groups.

### View Plan Status

<p>
<img alt="Plan Status" src="../../img/ui_plan_execution_plan_status.png" />
</p>

1. Follow the steps in ["Search and Select A Plan" section](#search-and-select-a-plan) to select a plan.
2. Click the "Status" tab if it is not the current tab. 
3. Use the settings here to alter the results listed in the plan history. By default, only the 5 most recent instances will be listed.  

     - "Show n records": controls the maximum number of recent plan instances to display.  
     - "Refresh every 60 secs": controls whether to automatically refresh the plan history.  
     - "Refresh": causes a reload of plan history. 
    
4. Select the desired plan instance. 
5. There are a few things you can do here.
 
    - "Refresh": updates the content of the result plan with the latest status of the plan instance.
    - "Cancel": cancels the executing plan.  
    - "Show Diagram": gives you a diagram view of the result plan.  
    - "A-", "A", "A+" buttons lets you alter the font size.  
    

### Cancel Plan

1. Follow the instructions in ["View Plan Status" section](#view-plan-status).
2. Click the "Cancel" button located right above the Result Plan.

