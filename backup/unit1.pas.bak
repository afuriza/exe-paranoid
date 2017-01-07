unit Unit1;

{$mode objfpc}{$H+}

interface

uses
  Classes, SysUtils, FileUtil, Forms, Controls, Graphics, Dialogs, StdCtrls,
  ExtCtrls, ComCtrls, Windows, ComObj, ActiveX, Variants, Unit2, jwapsapi, ShellAPI,
  jwaTlHelp32;

type

  { TForm1 }

  TForm1 = class(TForm)
    Button1: TButton;
    Button2: TButton;
    Memo1: TMemo;
    Timer1: TTimer;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure FormShow(Sender: TObject);
    procedure Timer1Timer(Sender: TObject);
  private
    { private declarations }
  public
    { public declarations }
  end;

  THook = class(TThread)
  private
    pstd, cmds, pids: string;
    //ends: boolean;
    procedure Refresh;
    procedure GetWin32_InstanceCreationEvent;
    function VarStrNUll(VarStr: olevariant): string;
    function GetWMIObject(const objectName: string): IDispatch;
  protected
    procedure Execute; override;
  public
    constructor Create;
  end;

var
  Form1: TForm1;
  hook: THook;
  tmcountdown: integer = 3;

implementation

{$R *.lfm}

function TerminateProcessByID(ProcessID: cardinal): boolean;
var
  hProcess: THandle;
begin
  Result := False;
  hProcess := OpenProcess(PROCESS_TERMINATE, False, ProcessID);
  if hProcess > 0 then
    try
      Result := Win32Check(Windows.TerminateProcess(hProcess, 0));
    finally
      CloseHandle(hProcess);
    end;
end;

function THook.VarStrNUll(VarStr: olevariant): string;
  //dummy function to handle null variants
begin
  Result := '';
  if not VarIsNull(VarStr) then
    Result := VarToStr(VarStr);
end;

function Thook.GetWMIObject(const objectName: string): IDispatch;
  //create a wmi object instance
var
  chEaten: PULong;
  BindCtx: IBindCtx;
  Moniker: IMoniker;
begin
  OleCheck(CreateBindCtx(0, bindCtx));
  OleCheck(MkParseDisplayName(BindCtx, StringToOleStr(objectName), chEaten, Moniker));
  OleCheck(Moniker.BindToObject(BindCtx, nil, IDispatch, Result));
end;

procedure THook.GetWin32_InstanceCreationEvent;
var
  objWMIService: olevariant;
  colMonitoredProcesses: olevariant;
  objLatestProcess: olevariant;
begin
  objWMIService := GetWMIObject('winmgmts:\\localhost\root\cimv2');
  colMonitoredProcesses :=
    objWMIService.ExecNotificationQuery(
    'Select * From __InstanceCreationEvent Within 1 Where TargetInstance ISA ''Win32_Process''');
  //Get the event listener
  while not Terminated do
  begin
    try
      objLatestProcess := colMonitoredProcesses.NextEvent(100);
      //set the max time to wait (ms)
    except
      on E: EOleException do
        if EOleException(E).ErrorCode = HRESULT($80043001) then
          //Check for the timeout error wbemErrTimedOut 0x80043001
          objLatestProcess := Null
        else
          raise;
    end;

    if not VarIsNull(objLatestProcess) then
    begin
      try
        pstd := VarStrNUll(objLatestProcess.TargetInstance.Name);
        cmds := VarStrNUll(objLatestProcess.TargetInstance.CommandLine);
        pids := VarStrNUll(objLatestProcess.TargetInstance.ProcessID);
        Synchronize(@Refresh);
      except
      end;
    end;
  end;
end;

{ TForm1 }

procedure THook.Refresh;
var
  item: TListItem;
  a: TDateTime;
  i: integer;
begin
  if (SysUtils.RightStr(cmds, 1) = '') or (SysUtils.RightStr(cmds, 21) =
    '"C:\Windows\System32\') or (UpperCase(SysUtils.RightStr(cmds, 20)) =
    SysUtils.UpperCase('C:\Windows\System32\')) or
    (UpperCase(SysUtils.RightStr(cmds, 20)) = SysUtils.UpperCase(
    'C:\Windows\SysWOW64\')) then
  else
  begin
    Unit1.TerminateProcessByID(StrToInt(PIDS));
    if not (Form2.ListView1.Items.Count = 0) then
      for i := 0 to Form2.ListView1.Items.Count - 1 do
      begin
        if not (Form2.ListView1.Items[i].SubItems.Strings[0] = cmds) then
        begin
          a := Now;
          Item := Form2.ListView1.Items.Add;
          Item.Caption := DateTimeToStr(a);
          Item.SubItems.Add(cmds);
        end;
      end
    else
    begin
      a := Now;
      Item := Form2.ListView1.Items.Add;
      Item.Caption := DateTimeToStr(a);
      Item.SubItems.Add(cmds);
    end;
    if Form2.Visible = False then
    begin
      try
        Form2.aflockfile(SysUtils.StringReplace(cmds, '"', '', [rfReplaceAll]), True);
      except
      end;
      Form2.Show;
      Form2.Label1.Caption := cmds;
    end;
  end;
  Form1.Memo1.Lines.Add('PID: ' + PIDS);
  Form1.Memo1.Lines.Add('CommandLine: ' + Cmds);
  Form1.Memo1.Lines.Add('Process: ' + pstd);
end;

constructor THook.Create;
begin
  inherited Create(True);
end;

procedure THook.Execute;
begin
  try
    CoInitialize(nil);
    try
      GetWin32_InstanceCreationEvent;
    finally
      CoUninitialize;
    end;

  except
    on E: Exception do
    begin
      Form1.Memo1.Lines.Add(E.ClassName + ': ' + E.Message);
      Form1.Timer1.Enabled := True;
    end;
  end;
end;

procedure TForm1.FormShow(Sender: TObject);
begin
  ShowMessage('Warning!' + #13 +
    'Please close this program when you don''t know what are you doing, ' +
    'this program is not fully stable. This program may destroy your house and kill your kitten.'
    + #13 + #13 + '* Here be dragons! Safe your data first.' + #13 + #13 +
    #13 + #13 + #13 + '~ Dio Affriza - AfsLab development and extreme research.' +
    #13 + 'Enjoy to my experiments. ;)');
end;

procedure TForm1.Timer1Timer(Sender: TObject);
begin
  Memo1.Lines.Add('Restarting...');
  if tmcountdown = 3 then
    tmcountdown := 2
  else if tmcountdown = 2 then
    tmcountdown := 1
  else if tmcountdown = 1 then
    tmcountdown := 0
  else if tmcountdown = 0 then
  begin
    tmcountdown := 3;
    Timer1.Enabled := False;
    Button2.Click;
    Button1.Click;
  end;
end;

procedure TForm1.Button1Click(Sender: TObject);
begin
  Button1.Enabled := False;
  Form1.Memo1.Lines.Add('Affriza Exe Paranoid start.');
  hook := THook.Create;
  hook.Start;
  Button2.Enabled := True;
end;

procedure TForm1.Button2Click(Sender: TObject);
begin
  Button2.Enabled := False;
  hook.Terminate;
  hook.Free;
  Button1.Enabled := True;
  Form1.Memo1.Lines.Add('Affriza Exe Paranoid suspended.');
end;

end.
