<div align="center">

## Stealth


</div>

### Description

Hide your porgams in the task manager, and from process lists.

No one will ever know its there!
 
### More Info
 
This control does not work under Windows 2000 as the tasking is different, I use it on win 95, 98, ME systems and it works great!


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Dave Bayliss](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/dave-bayliss.md)
**Level**          |Intermediate
**User Rating**    |5.0 (20 globes from 4 users)
**Compatibility**  |Delphi 5
**Category**       |[Custom Controls/ Forms/ Menus](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/custom-controls-forms-menus__7-4.md)
**World**          |[Delphi](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/delphi.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/dave-bayliss-stealth__7-420/archive/master.zip)

### API Declarations

```
If you Like it then vote.
If you dont like it then tell me how to make it better!
```


### Source Code

```
{Created By David Bayliss http://www.dbayliss.com}
unit Stealth;
interface
uses
 WinTypes, WinProcs, Classes, Forms, SysUtils, Controls, Messages;
type
 TDuplicateComponent = class(Exception);
 TFormNotOwner = class(Exception);
 TStealth = class(TComponent)
 private
  FHideForm: Boolean;
  fHideApp: Boolean;
  OldWndProc: TFarProc;
  NewWndProc: Pointer;
  function IsIt: Boolean;
  procedure SetIt (Value: Boolean);
  procedure SetHideApp(Value: Boolean);
  procedure HookParent;
  procedure UnhookParent;
  procedure HookWndProc(var Message: TMessage);
 protected
  { Protected declarations }
  procedure HideApplication;
  procedure ShowApplication;
 public
  { Public declarations }
  constructor Create(AOwner: TComponent); override;
  destructor Destroy; override;
  procedure Loaded; override;
  procedure ProcessEnabled;
 published
  { Published declarations }
  property HideForm: Boolean read IsIt write SetIt stored true default true;
  property HideApp: Boolean read fHideApp write SetHideApp;
 end;
function RegisterServiceProcess(dwProcessID, dwType: Integer): Integer; stdcall; external 'KERNEL32.DLL';
procedure Register;
implementation
destructor TStealth.Destroy;
begin
 ShowApplication;
 UnhookParent;
 inherited destroy;
end;
constructor TStealth.Create(AOwner: TComponent);
var
 i: Word;
 CompCount: Byte;
begin
 inherited Create(AOwner);
 fHideform := true;
 NewWndProc := nil;
 OldWndProc := nil;
 CompCount := 0;
 if (csDesigning in ComponentState) then
  if (AOwner is TForm) then
   with (AOwner as TForm) do
   begin
    for i := 0 to ComponentCount - 1 do
     if Components[i] is TStealth then Inc(CompCount);
    if CompCount > 1 then raise TDuplicateComponent.Create('There is already a TStealth component on this Form');
   end
   else
    raise TFormNotOwner.Create('The owner of TStealth Component is not a TForm');
   HookParent;
end;
procedure TStealth.SetHideApp(Value: Boolean);
begin
 fHideApp := Value;
 if Value then
  HideApplication
 else
  ShowApplication;
end;
procedure TStealth.HideApplication;
begin
 if not (csDesigning in ComponentState) then
  RegisterServiceProcess(GetCurrentProcessID, 1);
end;
procedure TStealth.ShowApplication;
begin
 if not (csDesigning in ComponentState) then
  RegisterServiceProcess(GetCurrentProcessID, 0);
end;
procedure TStealth.Loaded;
begin
 inherited Loaded;           { Always call inherited Loaded method }
 if not (csDesigning in ComponentState) then
  ProcessEnabled;
end;
procedure TStealth.ProcessEnabled;
begin
 if not (csDesigning in ComponentState) then
  if fHideform then
   ShowWindow(FindWindow(nil, @Application.Title[1]), SW_HIDE)
  else
   ShowWindow(FindWindow(nil, @Application.Title[1]), SW_RESTORE);
end;
function TStealth.IsIt: Boolean;
begin
 Result := fHideform;
end;
procedure TStealth.SetIt(Value: Boolean);
begin
 fHideform := value;
 ProcessEnabled;
end;
procedure TStealth.HookParent;
begin
 if owner = nil then exit;
 OldWndProc := TFarProc(GetWindowLong((owner as TForm).Handle, GWL_WNDPROC));
 NewWndProc := MakeObjectInstance(HookWndProc);
 SetWindowLong((owner as TForm).Handle, GWL_WNDPROC, LongInt(NewWndProc));
end;
procedure TStealth.UnhookParent;
begin
 if (owner <> NIL) and Assigned(OldWndProc) then
  SetWindowLong((owner as TForm).Handle, GWL_WNDPROC, LongInt(OldWndProc));
 if Assigned(NewWndProc) then
  FreeObjectInstance(NewWndProc);
 NewWndProc := NIL;
 OldWndProc := NIL;
end;
procedure Register;
begin
 RegisterComponents('Dbayliss', [TStealth]);
end;
procedure TStealth.HookWndProc(var Message: TMessage);
begin
 if owner = NIL then exit;
  if (Message.Msg = WM_SHOWWINDOW) then
   if (Message.wParam <> 0) then
    ProcessEnabled;
 Message.Result := CallWindowProc(OldWndProc, (owner as TForm).Handle, Message.Msg, Message.wParam, Message.lParam);
end;
end.
```

