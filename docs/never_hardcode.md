[Next: Pathlib →](pathlib.md)

Beginners typically write code, that just runs now, but wont scale.
It is understandable; you have been wrestling with errors, and are just very happy that your code sort-of works. It works now, so, "if it works do not touch it", right?

Well, no. I will try to explain this with an example.
Let's assume your code looks like this:

```python
# explore_data.py
import pandas as pd


datafile = 'C:/Users/Jan/OneDrive/Documenten/MasterHU/opdrachten_Jan/data/processed/whatsapp-20240918-065128.csv'

df = pd.read_csv(datafile)
df.dtypes

df.head()
df.columns = df.columns.str.strip()
df['timestamp'] = pd.to_datetime(df['timestamp'])

df['jaar'] = df['timestamp'].dt.year
print(df.columns)
print(df['jaar'].head())
```
As a rule of thumb, everying that is a string here, is hardcoded...

So, yes, this will work. It won't crash. However, what if you ever change the structure of your folders? Your datafile will not be found. What if the `timestamp` column changes name? Are you going to look through all your code to find all the places where you used that column? I hope not...

In addition to that; this code loads the data. But, you will probably want to load (and maybe save) data a lot of times. So, it might be a good idea to write code for reading and writing files just once, and reuse it.

How could you rewrite this code without hardcoding? Well, it could look something like this:

```python
import pandas as pd
from pathlib import Path
from dataclasses import dataclass
from loguru import logger


@dataclass
class Settings:
    processed_dir: Path
    time_col: str

def get_processed_data(filename: Path, settings: Settings) -> pd.DataFrame:
    path = settings.processed_dir / filename
    if path.exists():
        logger.info(f"Reading data from {path}")
        return pd.read_csv(path)
    else:
        logger.error(f"File {path} does not exist")
        raise FileNotFoundError

def preprocess(df: pd.DataFrame, settings: Settings) -> pd.DataFrame:
    # remove leading and trailing whitespaces from column names
    df.columns = df.columns.str.strip()
    df[settings.time_col] = pd.to_datetime(df[settings.time_col])
    df['jaar'] = df[settings.time_col].dt.year
    logger.info(f"The columns present are: {df.columns}")
    return df

if __name__ == '__main__':
    settings = Settings(
        processed_dir=Path('data/processed').resolve(),
        time_col='timestamp'
    )
    datafile = 'whatsapp-20241021-090015.csv'
    df = get_processed_data(datafile, settings)
    df = preprocess(df, settings)
    logger.succes("Finished processing")
```

In contrast to the previous code, the refactored code will:

- run on everyones filesystem, not just on the current setup of Jan's laptop
- it will check if the path exists and print the path if the file is not found
- read the timecolumn as specified in the settings. If the column name every changes, you will have to change it at a single, central location.
- the 'jaar' is hardcoded; you could consider to centralize this as well, but in some situations you might leave it like it is, for example you know for sure this is only going to be used one time, in this function. This depends on context, and you might still need to decide to centalize this later on.
- you have split the code into different functions, and settings. This makes your code more robust against changes in your data etc.

In addition, you could look at this repository which shows how to avoid hard-coding: [github.com/sche34/click_example](https://github.com/sche34/click_example). It's a pipeline that creates one or two visuals, similar to the assignments in (MADS-)DAV but with dummy data. It has a central configuration file (config.toml) with sections for every visual that allows you to change the (axis) title and configure the visual. In addition, it uses an abstract class that avoids the use of duplicate code and a standardized structure of the class. For more information about the use of abstract classes take a look at [abstract_code](abstract_code.md).

[Next: Pathlib →](pathlib.md)
