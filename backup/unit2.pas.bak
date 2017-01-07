unit Unit2;

{$mode objfpc}{$H+}

interface

uses
  Classes, SysUtils, FileUtil, Forms, Controls, Graphics, Dialogs, StdCtrls,
  ComCtrls, Windows, Process;

type

  { TForm2 }

  TForm2 = class(TForm)
    Button1: TButton;
    Button2: TButton;
    Label1: TLabel;
    Label2: TLabel;
    ListView1: TListView;
    procedure Button1Click(Sender: TObject);
    procedure Button2Click(Sender: TObject);
    procedure FormClose(Sender: TObject; var CloseAction: TCloseAction);
    procedure Label2Click(Sender: TObject);
    procedure ListView1Change(Sender: TObject; Item: TListItem;
      Change: TItemChange);
    procedure ListView1SelectItem(Sender: TObject; Item: TListItem;
      Selected: boolean);
  private
    { private declarations }
  public
    { public declarations }
    procedure aflockfile(const f: string; b: boolean);
  end;

var
  Form2: TForm2;
  aHandle: THandle;

implementation

uses Unit1;

{$R *.lfm}

{ TForm2 }

procedure TForm2.aflockfile(const f: string; b: boolean);
var
  aFileSize: integer;
  aFileName: string;
begin
  aFileName := f;
  aHandle := CreateFile(PChar(aFileName), GENERIC_READ, 0, nil,
    OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0); // get the handle of the file
  try
    aFileSize := GetFileSize(aHandle, nil);
    //get the file size for use in the  lockfile function
    if b = True then
      Win32Check(LockFile(aHandle, 0, 0, aFileSize, 0)) //lock the file
    //your code



    else
    begin
      //Win32Check(UnlockFile(aHandle,0,0,aFileSize,0));
      //unlock the file
      CloseHandle(aHandle);//Close the handle of the file.
    end;
  finally
    //CloseHandle(aHandle);//Close the handle of the file.
  end;
end;

procedure TForm2.Label2Click(Sender: TObject);
begin

end;

procedure TForm2.ListView1Change(Sender: TObject; Item: TListItem; Change: TItemChange);
begin

end;

procedure TForm2.ListView1SelectItem(Sender: TObject; Item: TListItem;
  Selected: boolean);
begin
  try
    Label1.Caption := ListView1.Selected.SubItems.Strings[0];
  except
  end;
end;

procedure TForm2.FormClose(Sender: TObject; var CloseAction: TCloseAction);
begin
  CloseHandle(aHandle);
end;

procedure TForm2.Button1Click(Sender: TObject);
var
  RunProgram: TProcess;
begin
  Form1.Button2.Click;
  try
    CloseHandle(aHandle);
    RunProgram := TProcess.Create(nil);
    RunProgram.CommandLine := ListView1.Selected.SubItems.Strings[0];
    RunProgram.Execute;
    RunProgram.Free;
    try
      ListView1.Selected.Delete;
    except
      Label1.Caption := 'Select some item';
    end;
  except
    Form1.Memo1.Lines.Add('Warning: Couldn''t execute process!');
  end;
  Form1.Button1.Click;
  if ListView1.Items.Count = 0 then
    Form2.Hide;
end;

procedure TForm2.Button2Click(Sender: TObject);
begin
  ListView1.Clear;
  Form2.Close;
end;

end.
