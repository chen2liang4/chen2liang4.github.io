---
title: Use dictionary to eliminate duplicate and get better maintenance
---

Below is a method in PTO app code. Hard code is not an issue in this small project, otherwise it’s maybe better for reading. But we can see duplicated code. Above 10 case statements are same, except parameters.
```c#
private void UpdateTimeBank(string leaveType, string hours, SPListItem timeBank)
{
    switch (leaveType)
    {
        case "Annual Leave":
            {
                UpdateSingleTimeBankItem(hours, "AnnualLeaveAvailable", "AnnualLeaveApproved", timeBank);
                break;
            }
        case "SickLeave with Pay":
            {
                UpdateSingleTimeBankItem(hours, "SickLeavewithPayAvailable", "SickLeaveWithPayApproved", timeBank);
                break;
            }
        // Here are 9 more similiar case statements.
        case "Bereavement Leave":
            {
                UpdateSingleTimeBankItem(hours, "BereavementLeaveAvailable", "BereavementLeaveApproved", timeBank);
                break;
            }
    }
}
```
I’d like to re-factor it without complex implement, like import extra class or resource.

## Use array
```c#
string[,] leaveTypes = {
                        {"Annual Leave", "AnnualLeaveAvailable", "AnnualLeaveApproved"},
                        {"SickLeave with Pay", "SickLeavewithPayAvailable", "SickLeaveWithPayApproved"},
                        {"SwapOT", "SwapOTAvailable", "SwapOTApproved"}
                       };
for (var i = 0; i < leaveTypes.GetLength(0); i++)
{
    if (leaveTypes[i, 0] == leaveType)
    {
        UpdateSingleTimeBankItem(hours, leaveTypes[i, 1], leaveTypes[i, 2], timeBank);
        break;
    }
}
```  
## Use anonymous class and collection
```c#
 var leaveTypeAnon = new [] {new {LeaveType="Annual Leave", AvailbleField="AnnualLeaveAvailable", ApprovedField="AnnualLeaveApproved"},
                             new {LeaveType="Sick Leave with Pay", AvailbleField="SickLeavewithPayAvailable", ApprovedField="SickLeaveWithPayApproved"},
                             new {LeaveType="Swap OT", AvailbleField="SwapOTAvailable", ApprovedField="SwapOTApproved"},};
 var target = leaveTypeAnon.Where(item => item.LeaveType == leaveType).First();
 if (null != target)
 {
     UpdateSingleTimeBankItem(hours, target.LeaveType, target.AvailbleField, target.ApprovedField, timeBank);
 }
```
We also can use Dictionary(string, List<string>), or List<List<string> to story leave type data. These ways are simple and plain than introducing one LeaveType class.