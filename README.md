Select *
From dbo.CovidDeaths
Where continent is not null
order by 3,4

--Select *
--From dbo.CovidVaccinations
--order by 3,4

--Select Data that we're going to be using

Select Location, date, total_cases,new_cases,total_deaths,population
From dbo.CovidDeaths
order by 1,2

--Looking at the Total Cases vs Total Deaths

Select Location, date, total_cases,total_deaths,(total_deaths/total_cases)as DeathPercentage
From dbo.CovidDeaths
Where location like '%states%'
order by 1,2


--We needed DeathPercentage to have values greater than 0(In decimal places)
--Shows Likelihood of dying if you contract covid in your country
Select location, date, total_cases,total_deaths, 
(CONVERT(float, total_deaths) / NULLIF(CONVERT(float, total_cases), 0)) * 100 AS Deathpercentage
from dbo.CovidDeaths
Where location like 'Kenya'
order by 1,2

--Looking at Total Cases vs Population
Select location, date, total_cases,population, 
(CONVERT(float, total_cases) / NULLIF(CONVERT(float, population), 0)) * 100 AS Deathpercentage
from dbo.CovidDeaths
Where location like 'Kenya'
order by 1,2

--Looking at countries with Highest Infection Rate compared to Population
Select Location, Population, MAX(total_cases) as HighestInfectionCount,
Max((CONVERT(float, total_cases) / NULLIF(CONVERT(float, population), 0)))* 100 AS PercentPopulationInfected
from dbo.CovidDeaths
group by Location,Population
order by PercentPopulationInfected desc

--Showing the countries with the Highest Death Count per Population
Select Location, MAX(total_deaths) as TotalDeathCount
From dbo.CovidDeaths
Where continent is not null
Group by Location
Order by TotalDeathCount desc



--Breaking things down by Continent

--Showing the continents with the highest death counts per population
Select Continent, MAX(total_deaths) as TotalDeathCount
From dbo.CovidDeaths
Where continent is not null
Group by continent
Order by TotalDeathCount desc


--Global Numbers

--1. To get the total cases vs the total deaths per day
Select date, SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths,
CONVERT(float,SUM(new_deaths))/SUM(New_cases)*100 as DeathPercentage
From dbo.CovidDeaths
Where continent is not null
Group by date
Order by 1,2

--2. To get %ge of the total cases vs the total deaths in the world
Select SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths,
CONVERT(float,SUM(new_deaths))/SUM(New_cases)*100 as DeathPercentage
From dbo.CovidDeaths
Where continent is not null
Order by 1,2



--Looking at Total Population vs Vaccinations

Select dea.continent, dea.location, dea.date, dea.population,vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location,dea.Date) as RollingPeopleVaccinated
--,(RollingPeopleVaccinated/Population)*100
From dbo.CovidDeaths dea
Join dbo.CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
Order by 2,3



--USE CTE
With PopvsVac (Continent,Location,Date,Population,New_Vaccinations,RollingPeopleVaccinated)
as
 (Select dea.continent, dea.location, dea.date, dea.population,vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location,dea.Date) as RollingPeopleVaccinated
--,(RollingPeopleVaccinated/Population)*100
From dbo.CovidDeaths dea
Join dbo.CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
--Order by 2,3
)
Select *,(CONVERT(float,RollingPeopleVaccinated)/CONVERT(float,Population))*100
From PopvsVac



--TEMP TABLE

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population,vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location,dea.Date) as RollingPeopleVaccinated
--,(RollingPeopleVaccinated/Population)*100
From dbo.CovidDeaths dea
Join dbo.CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--Where dea.continent is not null
--Order by 2,3
Select *,(CONVERT(float,RollingPeopleVaccinated)/CONVERT(float,Population))*100
From #PercentPopulationVaccinated


--Creating View to Store Data for Later Visualisations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population,vac.new_vaccinations
,SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.location Order by dea.location,dea.Date) as RollingPeopleVaccinated
--,(RollingPeopleVaccinated/Population)*100
From dbo.CovidDeaths dea
Join dbo.CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
--Order by 2,3


Select *
From PercentPopulationVaccinated
