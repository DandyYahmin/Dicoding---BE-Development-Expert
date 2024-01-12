const Rocket = require('./Rocket');
const RocketLauncher = require('./RocketLauncher');
const RocketRepairKit = require('./RocketRepairKit');

describe('A RocketLauncher', () => {
  it('should launch all rockets', () => {
    const nasaRocket = new Rocket('Nasa');
    const spaceXRocket = new Rocket('SpaceX');
    const rocketLauncher = new RocketLauncher([nasaRocket, spaceXRocket]);
    rocketLauncher.launchAllRockets();

    expect(nasaRocket.engineStatus).toEqual('active');
    expect(spaceXRocket.engineStatus).toEqual('active');
    expect(rocketLauncher.rockets.length).toEqual(0);
  });
  
  it('should launch only one rocket by queue', () => {
    const nasaRocket = new Rocket('Nasa');
    const spaceXRocket = new Rocket('SpaceX');
    const rocketLauncher = new RocketLauncher([nasaRocket, spaceXRocket]);
    rocketLauncher.launchRocketByQueue();

    expect(nasaRocket.engineStatus).toEqual('active');
    expect(spaceXRocket.engineStatus).toEqual('inactive');
    expect(rocketLauncher.rockets.length).toEqual(1);
  });

 //stub
  it('should return correct result when repair kit cannot repair the rocket', async () => {
    const fakeRocketRepairKit = {
      repair: () => Promise.reject('failed to repair the rocket'),
    };
    const rocketLauncher = new RocketLauncher(fakeRocketRepairKit, [{}]);
    const result = await rocketLauncher.repairAllRockets();

    expect(result).toEqual('there was 1 of 1 rocket fail to repair!');
  });

  //mock
  it('should repair some repairable rocket when repair kit cannot repair some the rocket', async () => {
    const repairableRocket = new Rocket('repairableRocket');
    const unrepairableRocket = new Rocket('unrepairableRocket');
    const fakeRocketRepairKit = {
      repair: jest.fn().mockImplementation((rocket) => {
        if (rocket.name === 'repairableRocket') {
          return Promise.resolve();
        }
        return Promise.reject('failed to repair the rocket');
      }),
    }
    const rocketLauncher = new RocketLauncher(fakeRocketRepairKit, [repairableRocket, unrepairableRocket]);
    const result = await rocketLauncher.repairAllRockets();

    expect(result).toEqual(`there was 1 of 2 rocket fail to repair!`);
    expect(fakeRocketRepairKit.repair).toBeCalled();
    expect(fakeRocketRepairKit.repair).toBeCalledWith(repairableRocket);
  });

  //spy
  it('should repair all the rockets with repair kit correctly', async () => {
    const nasaRocket = new Rocket('Nasa');
    const spaceXRocket = new Rocket('SpaceX');
    const rocketRepairKit = new RocketRepairKit({}, {}, {});
    const spyRepair = jest.spyOn(rocketRepairKit, 'repair');
    const rocketLauncher = new RocketLauncher(rocketRepairKit, [nasaRocket, spaceXRocket]);
    const result = await rocketLauncher.repairAllRockets();
  
    expect(spyRepair).toBeCalledTimes(2);
    expect(spyRepair).toBeCalledWith(nasaRocket);
    expect(spyRepair).toBeCalledWith(spaceXRocket);
    expect(result).toEqual('all rocket repaired!');
  });
});