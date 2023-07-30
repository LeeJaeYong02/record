Parser lib..  Copy https://gist.github.com/nsdevaraj/4620615

<details>
<summary>Parser lib.. <접기/펼치기></summary>
<div markdown="1">
    
 ### XMLReader.h

```
#import <Foundation/Foundation.h>

@interface XMLReader : NSObject <NSXMLParserDelegate>
{
    NSMutableArray *dictionaryStack;
    NSMutableString *textInProgress;
    NSError * __autoreleasing *errorPointer;
}

+ (NSDictionary *)dictionaryForPath:(NSString *)path error:(NSError **)errorPointer;
+ (NSDictionary *)dictionaryForXMLData:(NSData *)data error:(NSError **)errorPointer;
+ (NSDictionary *)dictionaryForXMLString:(NSString *)string error:(NSError **)errorPointer;

@end

@interface NSDictionary (XMLReaderNavigation)

- (id)retrieveForPath:(NSString *)navPath;

@end
```
	
### XMLReader.m
    
```
#import "XMLReader.h"

NSString *const kXMLReaderTextNodeKey = @"text";

@interface XMLReader (Internal)

- (id)initWithError:(NSError **)error;
- (NSDictionary *)objectWithData:(NSData *)data;

@end

@implementation NSDictionary (XMLReaderNavigation)

- (id)retrieveForPath:(NSString *)navPath
{
    // Split path on dots
    NSArray *pathItems = [navPath componentsSeparatedByString:@"."];
    
    // Enumerate through array
    NSEnumerator *e = [pathItems objectEnumerator];
    NSString *path;
    
    // Set first branch from self
    id branch = [self objectForKey:[e nextObject]];
    int count = 1;
    
    while ((path = [e nextObject]))
    {
        // Check if this branch is an NSArray
        if([branch isKindOfClass:[NSArray class]])
        {
            if ([path isEqualToString:@"last"])
            {
                branch = [branch lastObject];
            }
            else
            {
                if ([branch count] > [path intValue])
                {
                    branch = [branch objectAtIndex:[path intValue]];
                }
                else
                {
                    branch = nil;
                }
            }
        }
        else
        {
            // branch is assumed to be an NSDictionary
            branch = [branch objectForKey:path];
        }
        
        count++;
    }
    
    return branch;
}

@end

@implementation XMLReader

#pragma mark -
#pragma mark Public methods

+ (NSDictionary *)dictionaryForPath:(NSString *)path error:(NSError **)errorPointer
{
    NSString *fullpath = [[NSBundle bundleForClass:self] pathForResource:path ofType:@"xml"];
    NSData *data = [[NSFileManager defaultManager] contentsAtPath:fullpath];
    NSDictionary *rootDictionary = [XMLReader dictionaryForXMLData:data error:errorPointer];
    
    return rootDictionary;
}

+ (NSDictionary *)dictionaryForXMLData:(NSData *)data error:(NSError **)error
{
    XMLReader *reader = [[XMLReader alloc] initWithError:error];
    NSDictionary *rootDictionary = [reader objectWithData:data];
    
    return rootDictionary;
}

+ (NSDictionary *)dictionaryForXMLString:(NSString *)string error:(NSError **)error
{
    NSArray* lines = [string componentsSeparatedByString:@"\n"];
    NSMutableString* strData = [NSMutableString stringWithString:@""];

    for (int i = 0; i < [lines count]; i++)
    {
        [strData appendString:[[lines objectAtIndex:i] stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]]];
    }

    NSData *data = [strData dataUsingEncoding:NSUTF8StringEncoding];
    return [XMLReader dictionaryForXMLData:data error:error];
}

#pragma mark -
#pragma mark Parsing

- (id)initWithError:(NSError **)error
{
    if ((self = [super init]))
    {
        errorPointer = error;
    }
    
    return self;
}
 

- (NSDictionary *)objectWithData:(NSData *)data
{
    // Clear out any old data
    
    dictionaryStack = [[NSMutableArray alloc] init];
    textInProgress = [[NSMutableString alloc] init];
    
    // Initialize the stack with a fresh dictionary
    [dictionaryStack addObject:[NSMutableDictionary dictionary]];
    
    // Parse the XML
    NSXMLParser *parser = [[NSXMLParser alloc] initWithData:data];
    parser.delegate = self;
    BOOL success = [parser parse];
    
    // Return the stack's root dictionary on success
    if (success)
    {
        NSDictionary *resultDict = [dictionaryStack objectAtIndex:0];
        
        return resultDict;
    }
    
    return nil;
}

#pragma mark -
#pragma mark NSXMLParserDelegate methods

- (void)parser:(NSXMLParser *)parser didStartElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName attributes:(NSDictionary *)attributeDict
{
    // Get the dictionary for the current level in the stack
    NSMutableDictionary *parentDict = [dictionaryStack lastObject];
    
    // Create the child dictionary for the new element
    NSMutableDictionary *childDict = [NSMutableDictionary dictionary];

    // Initialize child dictionary with the attributes, prefixed with '@'
    for (NSString *key in attributeDict) {
        [childDict setValue:[attributeDict objectForKey:key]
                     forKey:[NSString stringWithFormat:@"@%@", key]];
    }
    
    // If there's already an item for this key, it means we need to create an array
    id existingValue = [parentDict objectForKey:elementName];
    
    if (existingValue)
    {
        NSMutableArray *array = nil;
        
        if ([existingValue isKindOfClass:[NSMutableArray class]])
        {
            // The array exists, so use it
            array = (NSMutableArray *) existingValue;
        }
        else
        {
            // Create an array if it doesn't exist
            array = [NSMutableArray array];
            [array addObject:existingValue];
            
            // Replace the child dictionary with an array of children dictionaries
            [parentDict setObject:array forKey:elementName];
        }
        
        // Add the new child dictionary to the array
        [array addObject:childDict];
    }
    else
    {
        // No existing value, so update the dictionary
        [parentDict setObject:childDict forKey:elementName];
    }
    
    // Update the stack
    [dictionaryStack addObject:childDict];
}

- (void)parser:(NSXMLParser *)parser didEndElement:(NSString *)elementName namespaceURI:(NSString *)namespaceURI qualifiedName:(NSString *)qName
{
    // Update the parent dict with text info
    NSMutableDictionary *dictInProgress = [dictionaryStack lastObject];
    
    // Pop the current dict
    [dictionaryStack removeLastObject];
    
    // Set the text property
    if ([textInProgress length] > 0)
    {
        if ([dictInProgress count] > 0)
        {
            [dictInProgress setObject:textInProgress forKey:kXMLReaderTextNodeKey];
        }
        else
        {
            // Given that there will only ever be a single value in this dictionary, let's replace the dictionary with a simple string.
            NSMutableDictionary *parentDict = [dictionaryStack lastObject];
            id parentObject = [parentDict objectForKey:elementName];
            
            // Parent is an Array
            if ([parentObject isKindOfClass:[NSArray class]])
            {
                [parentObject removeLastObject];
                [parentObject addObject:textInProgress];
            }
            
            // Parent is a Dictionary
            else
            {
                [parentDict removeObjectForKey:elementName];
                [parentDict setObject:textInProgress forKey:elementName];
            }
        }
         
        textInProgress = [[NSMutableString alloc] init];
    }
    
    // If there was no value for the tag, and no attribute, then remove it from the dictionary.
    else if ([dictInProgress count] == 0)
    {
        NSMutableDictionary *parentDict = [dictionaryStack lastObject];
        [parentDict removeObjectForKey:elementName];
    }
}

- (void)parser:(NSXMLParser *)parser foundCharacters:(NSString *)string
{
    // Build the text value
    [textInProgress appendString:[string stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]]];
}

- (void)parser:(NSXMLParser *)parser parseErrorOccurred:(NSError *)parseError
{
    // Set the error pointer to the parser's error object
    if (errorPointer)
        *errorPointer = parseError;
}

@end

```
	
</div>
</details>

### _Manifest

```
<keychains>
    <objects>
        <object name="IDMBID_INFO" />
        <object name="PUSHUUID_INFO" />
        <object name="UUID_INFO" />
    </objects>
</keychains>
```

### View.m

```
- (void) firstRunCheck {
    NSError *error = nil;
    NSString *path = [[NSBundle mainBundle] pathForResource:@"_Manifest" ofType:@"xml"];
    NSData *data = [NSData dataWithContentsOfFile:path];
    NSString *dataToStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSDictionary *_xmlDictionary = [XMLReader dictionaryForXMLString:dataToStr error:&error];
    NSMutableArray *subjectsArray = [[[_xmlDictionary objectForKey:@"keychains"] objectForKey:@"objects"] objectForKey:@"object"];
    
    if(![[[NSUserDefaults standardUserDefaults]valueForKey:@"firstRunFlag"] isEqual:@"false"]) {
        KeychainItemWrapper *keychain;
        for(int i=0; i<subjectsArray.count; i++) {
            NSDictionary *status = [subjectsArray objectAtIndex: i];
            NSLog(@"%@", [status objectForKey:@"@name"]);
            keychain = [[KeychainItemWrapper alloc] initWithIdentifier:[status objectForKey:@"@name"] accessGroup:nil];
            [keychain resetKeychainItem];
        }

        [[NSUserDefaults standardUserDefaults]setValue: @"false" forKey:@"firstRunFlag"];
    }
}
```
