#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <sstream>
#include <algorithm>
#include <memory>
#include <thread>
#include <mutex>
#include <map>
#include <set>
#include <list>
#include <vector>

template <typename T>
class Ingredient
{
public:
	Ingredient(std::string&& name, std::string&& color) :name_(name),
		color_(color) {}
	Ingredient(T& ref)
	{
		this->name_ = ref.applyName();
		this->color_ = ref.applyColor();
	}
	Ingredient(T&& rhs)
	{
		(*this).name_ = std::move(rhs.applyName());
		(*this).color_ = std::move(rhs.applyColor());
	}
	Ingredient(const Ingredient& ref)
	{
		(*this).name_ = ref.name_;
		(*this).color_ = ref.color_;
	}
	Ingredient& operator= (const Ingredient& ref)
	{
		(*this).name_ = ref.name_;
		(*this).color_ = ref.color_;
		return *this;
	}
	Ingredient(Ingredient&& rhs)
	{
		this->name_ = std::move(rhs.name_);
		this->name_ = std::move(rhs.color_);
	}
	~Ingredient() 
	{
		//simple cleanup -- protect you secret recipes by masking ingredient bits pour program post-mortem
		for (auto itr = name_.begin(); itr != name_.end(); ++itr)
		{
			(*itr) = 'X';
		}
		for (auto itr = color_.begin(); itr != color_.end(); ++itr)
		{
			(*itr) = 'X';
		}
	}
	//An ingredient is an ingredient, regardless of where it is defined--once is enough
	static void defineIngredient()
	{
		std::cout << "An Ingredient is any substance that goes into a recipe.\n"
			<< "Edible or not.\n";
	}
	void defineClassification()
	{
		static_cast<T*>(this)->defineClassification();
	}
	virtual void getClassification() const
	{
		std::cout << "No classification.";
	}
	std::string getName()
	{
		return this->name_;
	}
	std::string getColor()
	{
		return this->color_;
	}
protected:
	virtual std::string applyName() { return this->name_; }
	virtual std::string applyColor() { return this->color_; }
private:
	std::string name_;
	std::string color_;

};

class Animal :protected Ingredient<Animal>
{
public:
	Animal(std::string&& name, std::string&& color) :Ingredient(std::forward<std::string>(name), std::forward<std::string>(color))
	{
		
	}
	~Animal() {}
	Animal(const Ingredient<Animal>& ref) :Ingredient(ref) {}//down cast
	Animal(const Animal& ref) :Ingredient(ref)
	{
		(*this) = ref;
	}
	Animal& operator= (const Animal& ref) = default;
	Animal(Animal&& rhs) :Ingredient(rhs.applyName(), rhs.applyColor())
	{
		(*this) = std::move(rhs);
	}
	Animal& operator= (Animal&& rhs) = default;
	void defineClassification()
	{
		std::cout << "An animal is a multicellular heterotroph.\n";
	}
	virtual void getClassification()
	{
		std::cout << "ANIMAL";
	}
	virtual std::string applyName() { return Ingredient::applyName(); }
	virtual std::string applyColor() { return Ingredient::applyColor(); }
protected:

private:
};

class Plant :protected Ingredient<Plant>
{
public:
	Plant(std::string&& name, std::string&& color) :Ingredient(std::forward<std::string>(name), std::forward<std::string>(color))
	{
		
	}
	~Plant() {};
	Plant(const Ingredient<Plant>& ref) :Ingredient(ref) {}//down cast
	Plant(Animal& ref) :Ingredient(ref.applyName(), ref.applyColor()) {}//sideways cast
	Plant(Animal&& rhs) :Ingredient(rhs.applyName(), rhs.applyColor()){}
	Plant(const Plant& ref) :Ingredient(ref)
	{
		(*this) = ref;
	}
	Plant& operator= (const Plant& ref) = default;
	Plant(Plant&& rhs) :Ingredient(rhs.applyName(), rhs.applyColor())
	{
		(*this) = std::move(rhs);
	}
	Plant& operator= (Plant&& rhs) = default;
	void defineClassification()
	{
		std::cout << "A plant is a multicellular autotroph.\n";
	}
	virtual void getClassification()
	{
		std::cout << "PLANT";
	}
	virtual std::string applyName() { return Ingredient::applyName(); }
	virtual std::string applyColor() { return Ingredient::applyColor(); }
protected:

private:

};

//globals -- extern if declared outside
std::map<std::string, Ingredient<Animal>> animals;
std::map<std::string, Ingredient<Plant>> plants;
std::set<std::string> ingredients;
std::map<std::string, std::map<std::string, double>> recipes;
std::string animalString("animal");
std::string plantString("plant");
//thread creation condition
std::mutex readLock;//not needed unless like files are read simultaneously
std::mutex vectorPush;//wait to push into a shared container
//metric literals
constexpr long double operator""_kg(long double x) { return x * 0.001; }//kilogram
constexpr long double operator""_g(long double x) { return x; }//gram
constexpr long double operator""_dg(long double x) { return x * 10.0; }//decigram
constexpr long double operator""_cg(long double x) { return x * 100.0; }//centgram
constexpr long double operator""_mg(long double x) { return x * 1000.0; }//milligram


void readFile(const char* filePath, std::vector<std::string>& toParse)
{
	std::ifstream file(filePath);
	if (file.is_open())
	{
		while (!file.eof())
		{
			std::string tempString;
			std::getline(file, tempString);
			toParse.push_back(std::move(tempString));
		}
	}
	else
	{
		std::cout << "no file\n";
	}
	file.close();
}

void writeFile(const char* filePath, std::vector<std::string>& write)
{
	std::ofstream file(filePath);
	if (file.is_open())
	{
		for (std::vector<std::string>::iterator i = write.begin(); i != write.end(); ++i)
		{
			if (i == --(write.end()))
			{
				file << (*i);
			}
			else
			{
				file << (*i) << "\n";
			}
		}
	}
	file.close();
}

void setIngredient(std::string& type, std::string& name, std::string& color, std::vector<std::string>& data)
{
	if (type == std::string("") || type == std::string(" ") || type == std::string("\n") || type == std::string("\0"))
	{
		throw 1;
	}
	if (std::equal(type.begin(), type.end(), animalString.begin()))
	{
		if (!std::binary_search(ingredients.begin(), ingredients.end(), name))
		{
			//more efficient, and to the point then belows up_cast, however...
			//	these explicitly create child objects, then the child is cast into a parent when pushed to the map
			animals.insert(std::pair<std::string, Animal>(name, Animal(std::forward<std::string>(name), std::forward<std::string>(color))));
			//performed here to allow for type correctness to be eventually written out
			data.push_back(std::move(std::string(type + std::move(std::string(" ")) + name + std::move(std::string(" ")) + color)));
		}
	}
	else if (std::equal(type.begin(), type.end(), plantString.begin()))
	{
		if (!std::binary_search(ingredients.begin(), ingredients.end(), name))
		{
			//less efficient, merely to demonstrate moving more explicitly between oneself(happening anyway as pushing into pairs equivalent Plant type)
			plants.insert(std::pair<std::string, Plant>(name, Plant(std::forward<Plant>(Plant(std::forward<std::string>(name), std::forward<std::string>(color))))));
			//performed here to allow for type correctness to be eventually written out
			data.push_back(std::move(std::string(type + std::move(std::string(" ")) + name + std::move(std::string(" ")) + color)));
		}
	}
	else
	{
		throw 0;
	}

	std::pair<std::set<std::string>::iterator, bool> repeat = ingredients.insert(name);
	if (!repeat.second)
	{
		std::cout << "Duplicate ingredient: " << name << "\n";
	}
}

void setRecipe(const std::string& name, const std::string& filePath, const std::map<std::string, double>& ingr, std::vector<std::string>& dataFile, std::vector<std::vector<std::string>>& data)
{
	if (name == std::string("") || name == std::string(" ") || name == std::string("\n") || name == std::string("\0"))
	{
		throw 1;
	}
	auto success = recipes.insert(std::pair<std::string, std::map<std::string, double>>(name, ingr));
	if (success.second)
	{
		dataFile.push_back(std::move(std::string(name + std::move(std::string(" ")) + filePath)));
		std::vector<std::string> tempList;
		for (auto i = ingr.begin(); i != ingr.end(); ++i)
		{
			std::stringstream convert;
			convert << (*i).second;
			tempList.push_back(std::move(std::string((*i).first + std::move(std::string(" ")) + convert.str())));
		}
		data.push_back(std::move(tempList));
	}
}

class RecipeThread//functor to perform the recipe setting routine
{
	public:
		RecipeThread(std::vector<std::string>& dRFOut, std::vector<std::vector<std::string>>& dROut) :completeFlag(false)
		{
			dRF = &dRFOut;
			dR = &dROut;
		}
		~RecipeThread() = default;
		RecipeThread(const RecipeThread& ref): completeFlag(false)
		{
			this->dRF = ref.dRF;
			this->dR = ref.dR;
			this->completeFlag = ref.completeFlag;
			this->data = ref.data;
			this->filePath = ref.filePath;
			this->ingredientList = ref.ingredientList;
			this->name = ref.name;
		}
		RecipeThread(RecipeThread&& rhs): completeFlag(false)
		{
			this->dRF = rhs.dRF;
			this->dR = rhs.dR;
			this->completeFlag = std::move(rhs.completeFlag);
			this->data = std::move(rhs.data);
			this->filePath = std::move(rhs.filePath);
			this->ingredientList = std::move(rhs.ingredientList);
			this->name = std::move(rhs.name);
			//not really dynamic memory, but was a pointer that should not have privilages anymore
			rhs.dR = nullptr;
			rhs.dRF = nullptr;
		}
		RecipeThread& operator= (const RecipeThread& ref)
		{
			RecipeThread temp((*ref.dRF), (*ref.dR));
			temp.completeFlag = ref.completeFlag;
			temp.data = ref.data;
			temp.filePath = ref.filePath;
			temp.ingredientList = ref.ingredientList;
			temp.name = ref.name;
			return temp;
		}
		RecipeThread& operator= (RecipeThread&& rhs)
		{
			RecipeThread temp((*rhs.dRF), (*rhs.dR));
			temp.completeFlag = std::move(rhs.completeFlag);
			temp.data = std::move(rhs.data);
			temp.filePath = std::move(rhs.filePath);
			temp.ingredientList = std::move(rhs.ingredientList);
			temp.name = std::move(rhs.name);
			//not really dynamic memory, but was a pointer that should not have privilages anymore
			rhs.dR = nullptr;
			rhs.dRF = nullptr;
			return temp;
		}
		const bool& getCompleteFlag() const { return completeFlag; }
		void operator()(std::string recipeName, std::string files)
		{
			ingredientList.clear();
			data.clear();
			//scopes braces used to indicate separate security priorities when multi-threading, avoid parallel clashes(data race) -- extend with locks as needed
			{
				name = recipeName;
				filePath = files;
			}
			{
				//std::lock_guard<std::mutex> readingGuard(readLock);//only needed if same file being read, not locking unless known shared file exists
				::readFile(filePath.c_str(), data);
			}
			//safe concurrent codeblock, completely local
			for (std::vector<std::string>::iterator d = data.begin(); d != data.end(); ++d)
			{
				if ((*d) != std::string("") && (*d) != std::string("\n") && (*d) != std::string(" ") && (*d) != std::string("\0"))
				{
					std::stringstream tempStream((*d));
					std::string tName;
					double tQuantity;
					tempStream >> tName >> tQuantity;
					ingredientList.insert(std::pair<std::string, double>(tName, tQuantity));
				}
			}
			try
			{
				std::lock_guard<std::mutex> guard(vectorPush);
				::setRecipe(name, filePath, ingredientList, (*dRF), (*dR));
			}
			catch (...)
			{
				std::cout << "RECIPE ERROR\n";
			}
			this->completeFlag = true;
		}
	private:
		bool completeFlag;
		std::vector<std::string> data;
		std::vector<std::string>* dRF;
		std::vector<std::vector<std::string>> *dR;
		std::string filePath;
		std::map<std::string, double> ingredientList;
		std::string name;
};

struct colorTuple
{
	double blue;
	double red;
	double yellow;
	colorTuple operator+(const colorTuple& ref)
	{
		colorTuple temp;
		temp.blue = this->blue + ref.blue;
		temp.red = this->red + ref.red;
		temp.yellow = this->yellow + ref.yellow;
		return (temp);
	}
	colorTuple& operator+=(const colorTuple& ref)
	{
		this->blue += ref.blue;
		this->red += ref.red;
		this->yellow += ref.yellow;
		return (*this);
	}
	colorTuple operator* (const double& ref)
	{
		colorTuple temp;
		temp.blue = this->blue * ref;
		temp.red = this->red * ref;
		temp.yellow = this->yellow * ref;
		return temp;
	}
	friend colorTuple operator* (const double& ref, const colorTuple& refC)
	{
		colorTuple temp;
		temp.blue = refC.blue * ref;
		temp.red = refC.red * ref;
		temp.yellow = refC.yellow * ref;
		return temp;
	}
};

void analyzeColors(std::multimap<std::string, double>& ref, const double& mass)
{
	std::string black("black"), blue("blue"), green("green"), orange("orange"), pink("pink"), purple("purple"), red("red"), white("white"), yellow("yellow");
	colorTuple tBlack = { 1.0,1.0,1.0 }, tBlue = { 1.0,0.0,0.0 }, tGreen = { 1.0,0.0,1.0 }, tOrange = { 0.0,1.0,1.0, }, tPink = { 0.0,0.25,0.0 }, tPurple = { 1.0,1.0,0.0 },
		tRed = { 0.0,1.0,0.0 }, tWhite = { 0.0,0.0,0.0 }, tYellow = { 0.0,0.0,1.0 };
	std::string temp;
	colorTuple colorGuess = { 0.0,0.0,0.0 };
	bool notFound = false;
	std::vector<colorTuple> colorPalette;
	for (auto& pairs : ref)
	{
		temp = pairs.first;
		double ratio = pairs.second / mass;
		std::string::iterator itr;
		switch (temp.length())
		{
		default:
		{
			itr = std::search(temp.begin(), temp.end(), orange.begin(), orange.end());
			if (itr != temp.end())colorPalette.push_back(tOrange);

			itr = std::search(temp.begin(), temp.end(), purple.begin(), purple.end());
			if (itr != temp.end())colorPalette.push_back(tPurple);

			itr = std::search(temp.begin(), temp.end(), yellow.begin(), yellow.end());
			if (itr != temp.end())colorPalette.push_back(tYellow);
		}
		case 5:
		{
			itr = std::search(temp.begin(), temp.end(), black.begin(), black.end());
			if (itr != temp.end())colorPalette.push_back(tBlack);

			itr = std::search(temp.begin(), temp.end(), green.begin(), green.end());
			if (itr != temp.end())colorPalette.push_back(tGreen);

			itr = std::search(temp.begin(), temp.end(), white.begin(), white.end());
			if (itr != temp.end())colorPalette.push_back(tWhite);
		}
		case 4:
		{
			itr = std::search(temp.begin(), temp.end(), blue.begin(), blue.end());
			if (itr != temp.end())colorPalette.push_back(tBlue);

			itr = std::search(temp.begin(), temp.end(), pink.begin(), pink.end());
			if (itr != temp.end())colorPalette.push_back(tPink);
		}
		case 3:
		{
			itr = std::search(temp.begin(), temp.end(), red.begin(), red.end());
			if (itr != temp.end())colorPalette.push_back(tRed);
		}
		case 2:
		case 1:
		case 0:break;
		}
		if (colorPalette.size() > 0)
		{
			colorTuple tempMix = { 0.0,0.0,0.0 };
			double colorWeight = 1 / colorPalette.size();
			for (auto& i : colorPalette)
			{
				tempMix += colorWeight * i;
			}
			colorGuess += tempMix * ratio;
		}
		else
		{
			notFound = true;
		}
		colorPalette.clear();
	}
	int colors = -1;
	if (colorGuess.blue > colorGuess.red && colorGuess.blue > colorGuess.yellow)colors = 1;
	if (colorGuess.blue == colorGuess.red && colorGuess.blue > colorGuess.yellow)colors = 5;
	if (colorGuess.blue > colorGuess.red && colorGuess.blue == colorGuess.yellow)colors = 2;
	if (colorGuess.red > colorGuess.blue && colorGuess.red > colorGuess.yellow)colors = 6;
	if (colorGuess.red > colorGuess.blue && colorGuess.red == colorGuess.yellow)colors = 3;
	if (colorGuess.yellow > colorGuess.blue && colorGuess.yellow > colorGuess.red)colors = 8;
	if (colorGuess.blue == colorGuess.red && colorGuess.blue == colorGuess.yellow && colorGuess.red == colorGuess.yellow)
	{
		if (colorGuess.blue > 0)
		{
			colors = 0;
		}
		else
		{
			colors = 7;
		}
	}
	switch (colors)
	{
	case 0:
		{
			if (colorGuess.blue > 0.75)std::cout << "\nColor is dark and muddy\n";
			if (colorGuess.blue <= 0.75 && colorGuess.blue >= 0.5)std::cout << "\nColor is a mid-black possibly muddy\n";
			if (colorGuess.blue < 0.5)std::cout << "\nColor is a light black";
			break;
		}
	case 1:
		{
			if (0.4 <= (colorGuess.blue - colorGuess.red))std::cout << "\nColor is possibly a blueish purple\n";
			else if (0.4 <= (colorGuess.blue - colorGuess.yellow))std::cout << "\nColor is possibly a blueish green\n";
			else std::cout << "\nColor is blue\n";
			break;
		}
	case 2:
		{
			std::cout << "\nColor is green\n";
			break;
		}
	case 3:
		{
			std::cout << "\nColor is orange\n";
			break;
		}
	case 4:
		{
			//pink, faint ratios of colors ignored to avoid excessive inline code -- if extern less worry about bloat to sift through
			//	that is, with adequate space, color range determination could be extended
		}
	case 5:
		{
			std::cout << "\nColor is purple\n";
			break;
		}
	case 6:
		{
			if (0.4 <= (colorGuess.blue - colorGuess.red))std::cout << "\nColor is possibly a redish purple\n";
			else if (0.4 <= (colorGuess.red - colorGuess.yellow))std::cout << "\nColor is possibly a redish orange\n";
			else std::cout << "\nColor is red\n";
			break;
		}
	case 7:
		{
			std::cout << "\nColor is pale\n";
			break;
		}
	}
	if (notFound)std::cout << "\nSome colors could not be interpretted\n"
		<< "Color accuracy is further from reality\n\n";
}

template <typename T>
void getPounds(T val)
{
	static_assert(std::is_arithmetic<T>::value, "Not an Arithmetic type");//explicit catch post compiler template creation
	long double value = 0.0;
	value = static_cast<long double>(val);//this should throw compile-time error anyway
	std::cout << (value * 0.00220462) << "lb\n";
}

int main(int argc, char *argv[])
{
	const char* ingredientPath = "ingredients.txt";
	const char* recipePath = "recipes.txt";
	std::cout << "Welcome To Your Recipe Center!\n";

	std::vector<std::string> dataIn;
	std::vector<std::string> dataOut;
	readFile(ingredientPath, dataIn);
	if (dataIn.size() != 0)
	{
		for (auto i = dataIn.begin(); i != dataIn.end(); ++i)
		{
			std::istringstream tempStream((*i));
			std::string type, name, color;
			tempStream >> type >> name >> color;
			std::transform(type.begin(), type.end(), type.begin(), ::tolower);
			try
			{
				setIngredient(type, name, color, dataOut);
			}
			catch (...)
			{
				std::cout << "INCOMPATIBLE INGREDIENT: " << type << "\n";
			}
		}
	}

	std::vector<std::string> dataRIn;
	std::vector<std::string> dataRFOut;
	std::vector<std::vector<std::string>> dataROut;
	readFile(recipePath, dataRIn);
	if (dataRIn.size() != 0)
	{
		unsigned int availableThreads = std::thread::hardware_concurrency();
		std::vector<std::string> rNames, rFiles;
		if (availableThreads > 1)
		{
			for (auto i = dataRIn.cbegin(); i < dataRIn.cend(); ++i)
			{
				std::stringstream tempStream((*i));
				std::string tName, tFile;
				tempStream >> tName >> tFile;
				rNames.push_back(std::move(tName));
				rFiles.push_back(std::move(tFile));
			}
			std::list<std::thread> readingThreads;
			std::list<RecipeThread> recipeThreads;
			std::set<unsigned int> removeThreads;
			bool pendingQuota = false;
			int test = 0;
			while (rNames.size() != 0)
			{
				test += 1;
				if (readingThreads.size() >= (availableThreads - 1))
				{
					for (auto i = recipeThreads.begin(); i != recipeThreads.end(); ++i)
					{
						if ((*i).getCompleteFlag())
						{
							unsigned int value = std::distance(recipeThreads.begin(), i);
							removeThreads.insert(std::move(value));
						}
					}
					for (auto itr = removeThreads.rbegin(); itr != removeThreads.rend(); ++itr)//removing in reverse order as low to high would shift sequence, even with non-consequtive container
					{
						auto eItr = readingThreads.begin();
						std::advance(eItr, (*itr));
						readingThreads.erase(eItr);
						auto eItr2 = recipeThreads.begin();
						std::advance(eItr2, (*itr));
						recipeThreads.erase(eItr2);
					}
					removeThreads.clear();
				}
				else
				{
					recipeThreads.push_back((RecipeThread(dataRFOut, dataROut)));
					auto rItr = --recipeThreads.end();
					std::string tempFile = rFiles.back();
					std::string tempName = rNames.back();
					rFiles.pop_back();
					rNames.pop_back();
					readingThreads.push_back(std::move(std::thread(std::ref((*rItr)), tempName, tempFile)));
					auto launchItr = --readingThreads.end();
					launchItr->detach();
				}
				
			}
			removeThreads.clear();

			while (recipeThreads.size() != 0)
			{
				for (auto i = recipeThreads.begin(); i != recipeThreads.end(); ++i)
				{
					if ((*i).getCompleteFlag())
					{
						unsigned int value = std::distance(recipeThreads.begin(), i);
						removeThreads.insert(std::move(value));
					}
				}
				for (auto itr = removeThreads.rbegin(); itr != removeThreads.rend(); ++itr)//removing in reverse order as low to high would shift sequence, even with non-consequtive container
				{
					auto eItr = readingThreads.begin();
					std::advance(eItr, (*itr));
					readingThreads.erase(eItr);
					auto eItr2 = recipeThreads.begin();
					std::advance(eItr2, (*itr));
					recipeThreads.erase(eItr2);
				}
				removeThreads.clear();
			}

		}
		else
		{
			for (auto i = dataRIn.begin(); i != dataRIn.end(); ++i)
			{
				std::stringstream tempStream((*i));
				std::string name, filePath;
				tempStream >> name >> filePath;
				std::vector<std::string> tempIn;
				readFile(filePath.c_str(), tempIn);
				std::map<std::string, double> ingList;
				for (auto j = tempIn.begin(); j != tempIn.end(); ++j)
				{
					if ((*j) != std::string("") && (*j) != std::string("\n") && (*j) != std::string(" ") && (*j) != std::string("\0"))
					{
						std::stringstream tempStreamToo((*j));
						std::string ingName;
						double quantity;
						tempStreamToo >> ingName >> quantity;
						ingList.insert(std::pair<std::string, double>(ingName, quantity));
					}
				}
				try
				{
					setRecipe(name, filePath, ingList, dataRFOut, dataROut);
				}
				catch (...)
				{
					std::cout << "RECIPE ERROR!\n";
				}
			}
		}
	}

	bool quit = false;
	bool softQuit = false;
	char input = '0';
	std::string inputString;
	std::string inputStringTwo;
	std::string inputStringThree;
	double quan;
	std::map<std::string, double> tempIngList;
	while (!quit)
	{
		std::cout << "\n\n\nAdd Ingredient(i) Add Recipe(r) Check Ingredients(I) Check Recipes(R)\nQuit(q)\n";
		std::cin >> input;
		switch (input)
		{
		case 'i':
			std::cout << "Insert type: animal or plant\n";
			std::cin >> inputString;
			std::cout << "Insert name\n";
			std::cin >> inputStringTwo;
			std::cout << "Insert color\n";
			std::cin >> inputStringThree;
			try
			{
				setIngredient(inputString, inputStringTwo, inputStringThree, dataOut);
			}
			catch (...)
			{
				std::cout << "Ingredient creation failed\n";
			}
			break;
		case 'r':
			std::cout << "Insert recipe name\n";
			std::cin >> inputString;
			inputStringThree = inputString + std::string(".txt");
			while (!softQuit)
			{
				char inputTwo = '0';
				std::cout << "Insert ingredient\n";
				std::cin >> inputStringTwo;
				bool pending = false;
				if (std::binary_search(ingredients.begin(), ingredients.end(), inputStringTwo))
				{
					pending = true;
				}
				else
				{
					std::map<int, std::string> suggestionList;
					for (auto i = ingredients.begin(); i != ingredients.end(); ++i)
					{
						int counter = 0;
						std::string tempString;
						bool found = false;
						if (inputStringTwo.size() == (*i).size())
						{
							tempString = inputStringTwo;
							found = true;
						}
						if (inputStringTwo.size() == (*i).size() - 1)
						{
							tempString = inputStringTwo + std::string("a");
							found = true;
						}
						if (inputStringTwo.size() == (*i).size() + 1)
						{
							tempString = inputStringTwo.substr(0, (inputStringTwo.size() - 1));
							found = true;
						}//range could be extended if desired
						if (found)//will fail if first character differs -- potentially better simple solution would be mismatch algorithm with predicate
						{
							std::equal((*i).begin(), (*i).end(), tempString.begin(), [&](char a, char b)->bool {if (::tolower(a) == ::tolower(b)) { counter += 1; return true; }return false; });
						}
						if (counter > 0)suggestionList.insert(std::pair<int, std::string>(counter, (*i)));
					}
					bool goodGuess = false;
					for (auto i = suggestionList.rbegin(); i != suggestionList.rend(); ++i)
					{
						std::cout << "Did you mean: " << (*i).second << "(y)\n";
						std::cin >> inputTwo;
						if (inputTwo == 'y' || inputTwo == 'Y')
						{
							inputStringTwo = (*i).second;
							goodGuess = true;
							pending = true;
							break;
						}
					}
					inputTwo = '0';
					if (!goodGuess)
					{
						std::cout << "Would you like to add: " << inputStringTwo << " to ingredients?(y)\n";
						std::cin >> inputTwo;
						if (inputTwo == 'y' || inputTwo == 'Y')
						{
							std::cout << "Insert type for " << inputStringTwo << ": plant or animal\n";
							std::string tempOne, tempTwo;
							std::cin >> tempOne;
							std::cout << "Insert color for " << inputStringTwo << "\n";
							std::cin >> tempTwo;
							try
							{
								setIngredient(tempOne, inputStringTwo, tempTwo, dataOut);
								pending = true;
							}
							catch (...)
							{
								std::cout << inputStringTwo << " failed to be added\n";
							}
						}
					}
					inputTwo = '0';
				}
				if (pending)
				{
					std::cout << "Insert a quantity in grams for: " << inputStringTwo << "(X.X)\n";
					std::cin >> quan;
					tempIngList.insert(std::pair<std::string, double>(inputStringTwo, quan));
				}
				std::cout << "More Ingredients?(y)\n";
				std::cin >> inputTwo;
				(inputTwo != 'y') ? softQuit = true : softQuit = false;
			}
			if (tempIngList.size() > 0)
			{
				try
				{
					setRecipe(inputString, inputStringThree, tempIngList, dataRFOut, dataROut);
				}
				catch (...)
				{
					std::cout << "Recipe Error Occurred; Recipe not added\n";
				}
			}
			else
			{
				std::cout << "No Ingredients, no Recipe\n";
			}
			tempIngList.clear();
			softQuit = false;
			break;
		case 'I':
			for (auto i = ingredients.begin(); i != ingredients.end(); ++i)
			{
				try
				{
					animals.at((*i));
					std::cout << (*i) << " is from an: ";
					Animal tempA = static_cast<Animal>(animals.at((*i)));//down cast to obtain correct virtual method
					tempA.getClassification();
					std::cout << " its color is: " << animals.at((*i)).getColor() << "\n";
				}
				catch (const std::out_of_range& err)
				{
					//std::cout << "OUTOFRANGE\n";
				}
				try
				{
					plants.at((*i));
					std::cout << (*i) << " is from a: ";
					Plant tempP = static_cast<Plant>(plants.at((*i)));//down cast for virtual method
					tempP.getClassification();
					std::cout << " its color is: " << plants.at((*i)).getColor() << "\n";
				}
				catch (const std::out_of_range& err)
				{
					//std::cout << "OUTOFRANGE\n";
				}
			}
			while (!softQuit)
			{
				char inputTwo = '0';
				std::cout << "\nGet Definition of: Ingredient(i) Animal(a) Plant(p) or"
					<< "\nDelete an Ingredient(d) or Compost an Animal(c)\nQuit Ingredient Check(q)";
				std::cin >> inputTwo;
				switch (inputTwo)
				{
				case 'i': {Ingredient<Animal>::defineIngredient(); break; }
				case 'a': {Ingredient<Animal> tempA(Animal(std::string("name"), std::string("color"))); tempA.defineClassification(); break; }
				case 'p': {Ingredient<Plant> tempP(Plant(std::string("name"), std::string("color"))); tempP.defineClassification(); break; }
				case 'd':
				{
					std::cout << "Enter the ingredient name to delete\n";
					std::cin >> inputString;
					auto itr = ingredients.find(inputString);
					if (itr != ingredients.end())
					{
						ingredients.erase(itr);
						auto itr2 = animals.find(inputString);
						auto itr3 = plants.find(inputString);
						if (itr2 != animals.end())
						{
							animals.erase(itr2);
						}
						if (itr3 != plants.end())
						{
							plants.erase(itr3);
						}
						auto modItr = dataOut.end();
						for (auto i = dataOut.begin(); i != dataOut.end(); ++i)
						{
							size_t width = (*i).find(inputString);
							if (width != std::string::npos)
							{
								//validate correct word
								std::string one, two, three;
								std::stringstream tempStream(*i);
								tempStream >> one >> two >> three;
								if (two == inputString)//found
								{
									modItr = i;
								}
							}
						}
						if (modItr != dataOut.end())
						{
							dataOut.erase(modItr);
						}
					}
					else
					{
						std::cout << "Ingredient to delete was not found\n";
					}
					break;
				}
				case 'c':
				{
					std::cout << "Enter an Animal name to convert to a Plant\n";
					std::cin >> inputString;
					auto itr = animals.find(inputString);
					if (itr != animals.end())
					{
						plants.insert(std::pair<std::string, Plant>(inputString, Plant(static_cast<Animal>((*itr).second))));//downcast to child, then sideways to sibling -- to dust
						animals.erase(itr);
						for (auto i = dataOut.begin(); i != dataOut.end(); ++i)
						{
							size_t width = (*i).find(inputString);
							if (width != std::string::npos)
							{
								//validate correct word
								std::string one, two, three;
								std::stringstream tempStream(*i);
								tempStream >> one >> two >> three;
								if (two == inputString)//found
								{
									one = std::string("plant");
									(*i) = std::string(std::move(one) + std::move(std::string(" ")) + std::move(two) + std::move(std::string(" ")) + three);
								}
							}
						}
					}
					else
					{
						std::cout << "Animal not found\n";
					}
				}
				case 'q':
				default:softQuit = true;
				}
			}
			softQuit = false;
			break;
		case 'R':
		{
			while (!softQuit)
			{
				char inputTwo = '0';
				for (auto i = recipes.cbegin(); i != recipes.cend(); ++i)
				{
					std::cout << (*i).first << "\n";
				}
				std::cout << "See Recipe(r) Gram Conversions Chart(g)\nQuit(q)\n";
				std::cin >> inputTwo;
				switch (inputTwo)
				{
				case 'r':
					{
						std::cout << "Insert a recipe to look at\n";
						std::cin >> inputString;
						auto itr = recipes.find(inputString);
						if (itr != recipes.end())
						{
							char inputThree = '0';
							std::cout << "Would you like units of mass in:\n"
								<< "kg(k) g(g) dg(d) cg(c) mg(m) or mixed to whole unit(w)\n";
							std::cin >> inputThree;
							std::vector<std::string> unknownIng;
							for (auto pitr : (*itr).second)
							{
								long double tempQuantity = pitr.second;
								std::cout << pitr.first << " ";
								auto existsItr = ingredients.find(pitr.first);
								if (existsItr == ingredients.end())
								{
									unknownIng.push_back(pitr.first);
								}
								switch (inputThree)
								{
								case 'k':std::cout << tempQuantity * 1.0_kg << "kg"; break;
								case 'd':std::cout << tempQuantity * 1.0_dg << "dg"; break;
								case 'c':std::cout << tempQuantity * 1.0_cg << "cg"; break;
								case 'm':std::cout << tempQuantity * 1.0_mg << "mg"; break;
								case 'w':
									{
										if (tempQuantity >= 1000.0)
										{
											std::cout << tempQuantity * 1.0_kg << "kg";
										}
										else if (tempQuantity < 1000.0 && tempQuantity >= 1.0)
										{
											std::cout << tempQuantity << "g";
										}
										else if (tempQuantity < 1.0 && tempQuantity >= 0.1)
										{
											std::cout << tempQuantity * 1.0_dg << "dg";
										}
										else if (tempQuantity < 0.1 && tempQuantity >= 0.01)
										{
											std::cout << tempQuantity * 1.0_cg << "cg";
										}
										else if (tempQuantity < 0.01)
										{
											std::cout << tempQuantity * 1.0_mg << "mg";
										}
										break;
									}
								case 'g':
								default:std::cout << tempQuantity << "g";
								}
								std::cout << "\n";
							}
							for (auto v : unknownIng)
							{
								std::cout << v << " is not a known ingredient would you like to add it?(y)\n";
								std::cin >> inputThree;
								if (inputThree == 'y' || inputThree == 'Y')
								{
									std::cout << "Insert type for " << v << ": plant or animal\n";
									std::cin >> inputStringTwo;
									std::cout << "Insert color for " << v << "\n";
									std::cin >> inputStringThree;
									try
									{
										setIngredient(inputStringTwo, v, inputStringThree, dataOut);
									}
									catch (...)
									{
										std::cout << inputStringTwo << " failed to be added\n";
									}
								}
							}
							std::cout << "Would you like a color guess for this recipe?(y)\n";
							std::cin >> inputThree;
							if (::tolower(inputThree) == 'y')
							{
								std::multimap<std::string, double> known;
								double mass = 0.0;
								for (auto& pitr : (*itr).second)
								{
									auto existsItr = ingredients.find(pitr.first);
									if (existsItr != ingredients.end())
									{
										auto ani = animals.find(pitr.first);
										auto pla = plants.find(pitr.first);
										if (ani != animals.end())
										{
											known.insert(std::pair<std::string, double>((*ani).second.getColor(), pitr.second));
											mass += pitr.second;
										}
										if (pla != plants.end())
										{
											known.insert(std::pair<std::string, double>((*pla).second.getColor(), pitr.second));
											mass += pitr.second;
										}
									}
									else
									{
										std::cout << "unknown ingredient " << pitr.first << " will be ignored\n";
									}
								}
								analyzeColors(known, mass);
							}
						}
						else
						{
							std::cout << "Recipe not found\n";
						}
						break;
					}
				case 'g':
					{
						std::cout << "\n1g to kg = " << 1.0_kg << "\n";
						std::cout << "1g to dg = " << 1.0_dg << "\n";
						std::cout << "1g to cg = " << 1.0_cg << "\n";
						std::cout << "1g to mg = " << 1.0_mg << "\n";
						char inputThree = '0';
						bool convert = false;
						std::cout << "Want to convert a value in grams to the other units?(y)";
						std::cin >> inputThree;
						(inputThree == 'y') ? convert = true : convert = false;
						if (convert)
						{
							std::cout << "Enter value in grams\n";
							long double grams = 0.0;
							std::cin >> grams;
							std::cout << grams << "g:\n";
							std::cout << "\t" << grams * 1.0_kg << "kg\n";
							std::cout << "\t" << grams * 1.0_dg << "dg\n";
							std::cout << "\t" << grams * 1.0_cg << "cg\n";
							std::cout << "\t" << grams * 1.0_mg << "mg\n";
							std::cout << "Want to convert " << grams << "g to pounds?(y)\n";
							std::cin >> inputThree;
							if (inputThree == 'y' || inputThree == 'Y')
							{
								getPounds<long double>((grams * 1.0_g));
							}
						}
						break;
					}
				case 'q':
				default:softQuit = true;
				}
			}
			softQuit = false;
			break;
		}
		case 'q':quit = true; break;
		default:std::cout << "Invalid Input\n\n\n"; break;
		}
	}

	//write back to file
	writeFile(ingredientPath, dataOut);
	writeFile(recipePath, dataRFOut);
	for (unsigned int i = 0; i < dataRFOut.size(); i++)
	{
		std::string rName, rFilePath;
		std::stringstream tempStream((dataRFOut[i]));
		tempStream >> rName >> rFilePath;
		writeFile(rFilePath.c_str(), dataROut[i]);
	}
	return 0;
}


