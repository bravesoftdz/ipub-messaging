# iPub Messaging
<img src="https://img.shields.io/static/v1?label=Delphi%20Supported%20Versions&message=XE7%2B&color=blueviolet&style=for-the-badge"> <img src="https://img.shields.io/static/v1?label=Supported%20platforms&message=Full%20Cross%20Platform&color=blue&style=for-the-badge">

Thread safe, asynchronous and simplistic messaging system for communication between classes / layers in delphi created by the iPub team.

## The problem
  Delphi has its own messaging system (System.Messaging.pas) that works well but is totally synchronous and thread unsafe. In multithreaded systems we always need to communicate with other classes, sometimes synchronously, sometimes asynchronously, sometimes synchronizing with the mainthread (in the case of the UI), and doing this without a message system (communicating directly) makes the code large and complex, prone to many bugs.

## The solution
  An ideal messaging system would be a thread safe system that would allow a class to subscribe and then unsubscribe to listen to a particular message over this time, and this listener class that will receiving the message is who will inform how the sender will invoke it's method: on the same thread (**posting**), on the main thread (**main**), on another thread (**async**) and on a thread other than main (**background** ). This is the basis of our messaging system, the usage is similar to another existing system, the Delphi Event Bus (DEB).

## Advantages over similar (DEB)
 - **Performance**: faster in all operations (subscribe, unsubscribe, post), in addition to 10x faster initialization
 - **Efficiency**: consumes half the memory
 - **Minor code**: lower generated binary and faster compilation
 
 See the comparison in an environment with 1000 objects:
|  | Subscribe | Post | Unsubscribe |
| --- | --- | --- | --- |
| iPub | 1.6368 ms | 1.3119 ms | 1.7666 ms |
| DEB | 9.8832 ms | 2.0293 ms | 4.0022 ms |

## Using
  #### Interface message
  
  ```delphi
  ILogOutMessage = interface
    ['{CA101646-B801-433D-B31A-ADF7F31AC59E}']
    // here you can put any data
  end;
  ```
    
  #### Listening to a message
  First, you must subscribe your class to listen to messages, then all public methods that have the [Subscribe] attribute will be subscribed.
  ```delphi
  TForm1 = class(TForm)
    procedure FormCreate(Sender: TObject);
    procedure FormDestroy(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
    [Subscribe(TipMessagingThread.Main)]
    procedure OnLogout(const AMessage: ILogOutMessage);
  end;
  
  ...
  
  procedure TForm1.FormCreate(Sender: TObject);
  begin
    GMessaging.Subscribe(Self);
  end;

  procedure TForm1.FormDestroy(Sender: TObject);
  begin
    GMessaging.Unsubscribe(Self);
  end;

  procedure TForm1.OnLogout(const AMessage: ILogOutMessage);
  begin
    Showmessage('Log out!');
  end;
  ```
  
  #### Sending a message
  ```delphi  
  var
    LMessage: ILogOutMessage
  begin
    LMessage := TLogOutMessage.Create;
    GMessaging.Post(LMessage);
  ```
  
  #### Other message types
  In the previous examples we have shown an interface message, but there are altogether 3 type of messages:
  | Identity | Parameter |
  | --- | --- |
  | name (explicit) | string |
  | guid of parameter interface (implicit) | interface |
  | guid of parameter interface (implicit) + name (explicit) | interface |

  To receive messages with name identity, just declare the name in method attribute:
  ```delphi  
    [Subscribe(TipMessagingThread.Main, 'Name')]
  ```
  To send messages with name identity, just inform it in the Post call:
  ```delphi  
    GMessaging.Post('Name', LMessage);
  ```
  Note: The explicit name is case-insensitive.
   
# License
The iPub Messaging is licensed under MIT, and the license file is included in this folder.