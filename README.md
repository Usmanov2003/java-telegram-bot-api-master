Java Telegram Bot API
Maven Central codecov

Java library for interacting with Telegram Bot API

Full support of all Bot API 7.2 methods
Telegram Passport and Decryption API
Bot Payments
Gaming Platform
Download
Gradle:

implementation 'com.github.pengrad:java-telegram-bot-api:7.4.0'
Maven:

<dependency>
  <groupId>com.github.pengrad</groupId>
  <artifactId>java-telegram-bot-api</artifactId>
  <version>7.4.0</version>
</dependency>
JAR with all dependencies on release page

Usage
// Create your bot passing the token received from @BotFather
TelegramBot bot = new TelegramBot("BOT_TOKEN");

// Register for updates
bot.setUpdatesListener(updates -> {
    // ... process updates
    // return id of last processed update or confirm them all
    return UpdatesListener.CONFIRMED_UPDATES_ALL;
// Create Exception Handler
}, e -> {
    if (e.response() != null) {
        // got bad response from telegram
        e.response().errorCode();
        e.response().description();
    } else {
        // probably network error
        e.printStackTrace();
    }
});

// Send messages
long chatId = update.message().chat().id();
SendResponse response = bot.execute(new SendMessage(chatId, "Hello!"));
Documentation
Creating your bot
Making requests
Getting updates
Available types
Available methods
Updating messages
Stickers
Inline mode
Payments
Telegram Passport
Games
Creating your bot
TelegramBot bot = new TelegramBot("BOT_TOKEN");
Network operations based on OkHttp library.
You can build bot with custom OkHttpClient, for specific timeouts or interceptors.

TelegramBot bot = new TelegramBot.Builder("BOT_TOKEN").okHttpClient(client).build();
Making requests
Synchronous

BaseResponse response = bot.execute(request);
Asynchronous

bot.execute(request, new Callback() {
    @Override
    public void onResponse(BaseRequest request, BaseResponse response) {
    
    }
    @Override
    public void onFailure(BaseRequest request, IOException e) {
    
    }
});
Request in response to update

String response = request.toWebhookResponse();
Getting updates
You can use getUpdates request, parse incoming Webhook request, or set listener to receive updates.
Update object just copies Telegram's response.

class Update {
    Integer updateId();
    Message message();
    Message editedMessage();
    InlineQuery inlineQuery();
    ChosenInlineResult chosenInlineResult();
    CallbackQuery callbackQuery();
}
Get updates
Building request

GetUpdates getUpdates = new GetUpdates().limit(100).offset(0).timeout(0);
The getUpdates method returns the earliest 100 unconfirmed updates. To confirm an update, use the offset parameter when calling getUpdates like this: offset = updateId of last processed update + 1
All updates with updateId less than offset will be marked as confirmed on the server and will no longer be returned.

Executing

// sync
GetUpdatesResponse updatesResponse = bot.execute(getUpdates);
List<Update> updates = updatesResponse.updates();
...
Message message = update.message()


// async
bot.execute(getUpdates, new Callback<GetUpdates, GetUpdatesResponse>() {
    @Override
    public void onResponse(GetUpdates request, GetUpdatesResponse response) {
        List<Update> updates = response.updates();
    }
    
    @Override
    public void onFailure(GetUpdates request, IOException e) {
    
    }
});
Webhook
Building request

SetWebhook request = new SetWebhook()
       .url("url")
       .certificate(new byte[]{}) // byte[]
       .certificate(new File("path")); // or file 
Executing

// sync
BaseResponse response = bot.execute(request);
boolean ok = response.isOk();

// async
bot.execute(request, new Callback<SetWebhook, BaseResponse>() {
    @Override
    public void onResponse(SetWebhook request, BaseResponse response) {
    
    }
    @Override
    public void onFailure(SetWebhook request, IOException e) {
        
    }
});
Using Webhook you can parse request to Update

Update update = BotUtils.parseUpdate(stringRequest); // from String
Update update = BotUtils.parseUpdate(reader); // or from java.io.Reader

Message message = update.message();
Updates Listener
You can set a listener to receive incoming updates as if using Webhook.
This will trigger executing getUpdates requests in a loop.

bot.setUpdatesListener(new UpdatesListener() {
    @Override
    public int process(List<Update> updates) {

        // process updates

        return UpdatesListener.CONFIRMED_UPDATES_ALL;
    }
// Create Exception Handler
}, new ExceptionHandler() {
    @override
    public void onException(TelegramException e)
    {
        if (e.response() != null) {
            // got bad response from telegram
            e.response().errorCode();
            e.response().description();
        } else {
            // probably network error
            e            .printStackTrace();
        }
    }
});
Listener should return id of the last processed (confirmed) update.
To confirm all updates return UpdatesListener.CONFIRMED_UPDATES_ALL, this should be enough in most cases.
To not confirm any updates return UpdatesListener.CONFIRMED_UPDATES_NONE, these updates will be redelivered.
To set a specific update as last confirmed, just return the required updateId.

To stop receiving updates

bot.removeGetUpdatesListener();
Available types
All types have the same name as original ones.
Type's fields are methods in lowerCamelCase.

Types used in responses (Update, Message, User, Document...) are in com.pengrad.telegrambot.model package.

Types used in requests (Keyboard, InlineQueryResult, ParseMode, InputMessageContent...) are in com.pengrad.telegrambot.model.request package.
When creating a request's type, required params should be passed in the constructor, optional params can be added in chains.

Keyboards
ForceReply, ReplyKeyboardRemove

Keyboard forceReply = new ForceReply(isSelective); // or just new ForceReply();
Keyboard replyKeyboardRemove = new ReplyKeyboardRemove(); // new ReplyKeyboardRemove(isSelective)
ReplyKeyboardMarkup

Keyboard replyKeyboardMarkup = new ReplyKeyboardMarkup(
                new String[]{"first row button1", "first row button2"},
                new String[]{"second row button1", "second row button2"})
                .oneTimeKeyboard(true)   // optional
                .resizeKeyboard(true)    // optional
                .selective(true);        // optional
KeyboardButton

Keyboard keyboard = new ReplyKeyboardMarkup(
        new KeyboardButton[]{
                new KeyboardButton("text"),
                new KeyboardButton("contact").requestContact(true),
                new KeyboardButton("location").requestLocation(true)
        }
);                
InlineKeyboardMarkup

InlineKeyboardMarkup inlineKeyboard = new InlineKeyboardMarkup(
        new InlineKeyboardButton[]{
                new InlineKeyboardButton("url").url("www.google.com"),
                new InlineKeyboardButton("callback_data").callbackData("callback_data"),
                new InlineKeyboardButton("Switch!").switchInlineQuery("switch_inline_query")
        });
Chat Action
ChatAction action = ChatAction.typing;
ChatAction action = ChatAction.upload_photo;
ChatAction action = ChatAction.find_location;
Available methods
All request methods have the same names as original ones.
Required params should be passed in the constructor.
Optional params can be added in chains.

Send message
All send requests (SendMessage, SendPhoto, SendLocation...) return SendResponse object that contains Message.

SendMessage request = new SendMessage(chatId, "text")
        .parseMode(ParseMode.HTML)
        .disableWebPagePreview(true)
        .disableNotification(true)
        .replyToMessageId(1)
        .replyMarkup(new ForceReply());

// sync
SendResponse sendResponse = bot.execute(request);
boolean ok = sendResponse.isOk();
Message message = sendResponse.message();

// async
bot.execute(request, new Callback<SendMessage, SendResponse>() {
    @Override
    public void onResponse(SendMessage request, SendResponse response) {
       
    }
    
    @Override
    public void onFailure(SendMessage request, IOException e) {
    
    }
});
Formatting options
ParseMode parseMode = ParseMode.Markdown;
ParseMode parseMode = ParseMode.HTML;
Get file
GetFile request = new GetFile("fileId")
GetFileResponse getFileResponse = bot.execute(request);

File file = getFileResponse.file(); // com.pengrad.telegrambot.model.File
file.fileId();
file.filePath();  // relative path
file.fileSize();
To get downloading link as https://api.telegram.org/file/<BOT_TOKEN>/<FILE_PATH>

String fullPath = bot.getFullFilePath(file);  // com.pengrad.telegrambot.model.File
Other requests
All requests return BaseResponse if not mention here

class BaseResponse {
  boolean isOk();
  int errorCode();
  String description();
}
GetMe request returns GetMeResponse

class GetMeResponse {
  User user();
}
GetChatAdministrators

class GetChatAdministratorsResponse {
  List<ChatMember> administrators()
}
GetChatMembersCount

class GetChatMembersCountResponse {
  int count() 
}
GetChatMember

class GetChatMemberResponse {
  ChatMember chatMember()
}
GetChat

class GetChatResponse {
  Chat chat()
}
GetUserProfilePhotos

class GetUserProfilePhotosResponse {
  UserProfilePhotos photos()
}
StopPoll

class PollResponse {
  Poll poll()
}
Updating messages
Normal message

EditMessageText editMessageText = new EditMessageText(chatId, messageId, "new test")
        .parseMode(ParseMode.HTML)
        .disableWebPagePreview(true)
        .replyMarkup(new ReplyKeyboardRemove());
        
BaseResponse response = bot.execute(editMessageText);
Inline message

EditMessageText editInlineMessageText = new EditMessageText(inlineMessageId, "new text");
BaseResponse response = bot.execute(editInlineMessageText);
Delete message

DeleteMessage deleteMessage = new DeleteMessage(chatId, messageId);
BaseResponse response = bot.execute(deleteMessage);
Stickers
Send sticker

// File or byte[] or string fileId of existing sticker or string URL
SendSticker sendSticker = new SendSticker(chatId, imageFile);
SendResponse response = bot.execute(sendSticker);
Get sticker set

GetStickerSet getStickerSet = new GetStickerSet(stickerSet);
GetStickerSetResponse response = bot.execute(getStickerSet);
StickerSet stickerSet = response.stickerSet();
Upload sticker file

// File or byte[] or string URL
UploadStickerFile uploadStickerFile = new UploadStickerFile(chatId, stickerFile);
GetFileResponse response = bot.execute(uploadStickerFile);
Inline mode
Getting updates

GetUpdatesResponse updatesResponse = bot.execute(new GetUpdates());
List<Update> updates = updatesResponse.updates();
...
InlineQuery inlineQuery = update.inlineQuery();
ChosenInlineResult chosenInlineResult = update.chosenInlineResult();
CallbackQuery callbackQuery = update.callbackQuery();
If using webhook, you can parse request to InlineQuery

Update update = BotUtils.parseUpdate(stringRequest); // from String
Update update = BotUtils.parseUpdate(reader); // from java.io.Reader

InlineQuery inlineQuery = update.inlineQuery();
Inline query result
InlineQueryResult r1 = new InlineQueryResultPhoto("id", "photoUrl", "thumbUrl");
InlineQueryResult r2 = new InlineQueryResultArticle("id", "title", "message text").thumbUrl("url");
InlineQueryResult r3 = new InlineQueryResultGif("id", "gifUrl", "thumbUrl");
InlineQueryResult r4 = new InlineQueryResultMpeg4Gif("id", "mpeg4Url", "thumbUrl");

InlineQueryResult r5 = new InlineQueryResultVideo(
  "id", "videoUrl", InlineQueryResultVideo.MIME_VIDEO_MP4, "message", "thumbUrl", "video title")
    .inputMessageContent(new InputLocationMessageContent(21.03f, 105.83f));
Answer inline query
BaseResponse response = bot.execute(new AnswerInlineQuery(inlineQuery.id(), r1, r2, r3, r4, r5));

// or full
bot.execute(
        new AnswerInlineQuery(inlineQuery.id(), new InlineQueryResult[]{r1, r2, r3, r4, r5})
                .cacheTime(cacheTime)
                .isPersonal(isPersonal)
                .nextOffset("offset")
                .switchPmParameter("pmParam")
                .switchPmText("pmText")
);
Payments
Send invoice

SendInvoice sendInvoice = new SendInvoice(chatId, "title", "desc", "my_payload",
        "providerToken", "my_start_param", "USD", new LabeledPrice("label", 200))
        .needPhoneNumber(true)
        .needShippingAddress(true)
        .isFlexible(true)
        .replyMarkup(new InlineKeyboardMarkup(new InlineKeyboardButton[]{
                new InlineKeyboardButton("just pay").pay(),
                new InlineKeyboardButton("google it").url("www.google.com")
        }));
SendResponse response = bot.execute(sendInvoice);
Answer shipping query

LabeledPrice[] prices = new LabeledPrice[]{
        new LabeledPrice("delivery", 100),
        new LabeledPrice("tips", 50)
};
AnswerShippingQuery answerShippingQuery = new AnswerShippingQuery(shippingQueryId,
        new ShippingOption("1", "VNPT", prices),
        new ShippingOption("2", "FREE", new LabeledPrice("free delivery", 0))
);
BaseResponse response = bot.execute(answerShippingQuery);

// answer with error
AnswerShippingQuery answerShippingError = new AnswerShippingQuery(id, "Can't deliver here!");
BaseResponse response = bot.execute(answerShippingError);
Answer pre-checkout query

AnswerPreCheckoutQuery answerCheckout = new AnswerPreCheckoutQuery(preCheckoutQueryId);
BaseResponse response = bot.execute(answerPreCheckoutQuery);

// answer with error
AnswerPreCheckoutQuery answerCheckout = new AnswerPreCheckoutQuery(id, "Sorry, item not available");
BaseResponse response = bot.execute(answerPreCheckoutQuery);
Telegram Passport
When the user confirms your request by pressing the ‘Authorize’ button, the Bot API sends an Update with the field passport_data to the bot that contains encrypted Telegram Passport data. Telegram Passport Manual

Receiving information
You can get encrypted Passport data from Update (via UpdatesListener or Webhook)

PassportData passportData = update.message().passportData();
PassportData contains anarray of EncryptedPassportElement and EncryptedCredentials.
You need to decrypt Credentials using private key (public key you uploaded to @BotFather)

String privateKey = "...";
EncryptedCredentials encryptedCredentials = passportData.credentials();
Credentials credentials = encryptedCredentials.decrypt(privateKey);
These Credentials can be used to decrypt encrypted data in EncryptedPassportElement.

EncryptedPassportElement[] encryptedPassportElements = passportData.data();
for (EncryptedPassportElement element : encryptedPassportElements) {
    DecryptedData decryptedData = element.decryptData(credentials);
    // DecryptedData can be cast to specific type by checking instanceOf 
    if (decryptedData instanceof PersonalDetails) {
        PersonalDetails personalDetails = (PersonalDetails) decryptedData;
    }
    // Or by checking type of passport element
    if (element.type() == EncryptedPassportElement.Type.address) {
        ResidentialAddress address = (ResidentialAddress) decryptedData;
    }
}
EncryptedPassportElement also contains an array of PassportFile (file uploaded to Telegram Passport).
You need to download them 1 by 1 and decrypt content.
This library supports downloading and decryption, returns decrypted byte[]

EncryptedPassportElement element = ...

// Combine all files 
List<PassportFile> files = new ArrayList<PassportFile>();
files.add(element.frontSide());
files.add(element.reverseSide());
files.add(element.selfie());
if (element.files() != null) {
    files.addAll(Arrays.asList(element.files()));
}
if (element.translation() != null) {
    files.addAll(Arrays.asList(element.translation()));
}

// Decrypt
for (PassportFile file : files) {
    if (file == null) continue;
    byte[] data = element.decryptFile(file, credentials, bot); // GetFile request and decrypt content
    // save to file if needed
    new FileOutputStream("files/" + element.type()).write(data);
}
Set Passport data errors
SetPassportDataErrors setPassportDataErrors = new SetPassportDataErrors(chatId,
        new PassportElementErrorDataField("personal_details", "first_name", "dataHash",
                "Please enter a valid First name"),
        new PassportElementErrorSelfie("driver_license", "fileHash",
                "Can't see your face on photo")
);
bot.execute(setPassportDataErrors);
Games
Send game

SendResponse response = bot.execute(new SendGame(chatId, "my_super_game"));
Set game score

BaseResponse response = bot.execute(new SetGameScore(userId, score, chatId, messageId));
Get game high scores

GetGameHighScoresResponse response = bot.execute(new GetGameHighScores(userId, chatId, messageId));
GameHighScore[] scores = response.result();
