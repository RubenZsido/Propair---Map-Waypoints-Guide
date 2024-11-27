# Map Waypoints Guide
How to add icons for important waypoints

The following guide is not complete, because half the code I had to use in the U2 project was included already. These were the steps I had to take to create the required functionality.

Add the following line to the MapSystemBuilder creation:

```js
 .withNearestWaypoints(MapSystemConfig.configureMapWaypoints(), false)
```
It should look something like this(the following code is from u2):
```js
this.mapSystem = MapSystemBuilder.create(this.props.bus)
      .withBing(`miltech-u2-map`, 0)
      .withNearestWaypoints(MapSystemConfig.configureMapWaypoints(), false)
      .withController(MapSystemKeys.TerrainColors, (context) => {
        this.mapTerrainColorsController = new MapTerrainColorsController(context);
        return this.mapTerrainColorsController;
      })
      .withController("offSet", (context) => new MapOffSetController(context))
      .withController("range", (context) => new MapRangeController(context))
      .withOwnAirplaneIcon(350, "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/plane-circles.svg", new Float64Array([0.5, 0.6]), "hsi-map-ownship-icon")
      .withRotation()
      .withOwnAirplaneIconOrientation(MapOwnAirplaneIconOrientation.HeadingUp)
      .withOwnAirplanePropBindings(["trackTrue", "position", "magVar"], 30) // trackTrue, hdgTrue
      .withFollowAirplane()
      .withProjectedSize(this.screenSize)
      .withClockUpdate(30)
      .build();
```
If it is missing, add the MapSystemConfig class
<details>

<summary>class MapSystemConfig</summary>

```js
   class MapSystemConfig {
  /**
   * Builds non-active leg style for hold legs.
   * @param vector The vector being rendered.
   * @param isIngress Whether or not this vector is an ingress vector.
   * @returns The appropriate hold leg display style.
   */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  static buildWhiteHoldStyle(vector, isIngress) {
    return MapSystemConfig.WhitePath;
  }
  /**
   * Builds active leg style for hold legs.
   * @param vector The vector being rendered.
   * @param isIngress Whether or not this vector is an ingress vector.
   * @returns The appropriate hold leg display style.
   */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  static buildMagentaHoldStyle(vector, isIngress) {
    return MapSystemConfig.MagentaPath;
  }
  /**
   * Builds leg style for hold legs on the missed approach.
   * @param vector The vector being rendered.
   * @param isIngress Whether or not this vector is an ingress vector.
   * @returns The appropriate hold leg display style.
   */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  static buildCyanHoldStyle(vector, isIngress) {
    return MapSystemConfig.CyanPath;
  }
  /**
   * Builds non-active leg style for hold legs.
   * @param vector The vector being rendered.
   * @param isIngress Whether or not this vector is an ingress vector.
   * @returns The appropriate hold leg display style.
   */
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  static buildWhiteDashedHoldStyle(vector, isIngress) {
    return MapSystemConfig.WhiteDashedPath;
  }
  /**
   * Gets the own airplane properties to bind to event bus events.
   * @returns An array of own airplane properties to bind to event bus events.
   */
  static getOwnAirplanePropsToBind() {
    return ["position", "hdgTrue", "trackTrue", "altitude", "verticalSpeed", "groundSpeed"];
  }
  /**
   * Builds a label for facility waypoints.
   * @param color The color of the label.
   * @returns A new factory that will create the label.
   */
  static buildFacilityLabel(color) {
    return (w) => {
      return new MapCullableLocationTextLabel(ICAO.getIdent(w.facility.get().icao), WT21MapWaypointIconPriority.Bottom, w.location, false, { fontSize: 32, fontColor: color, font: "WT21", anchor: new Float64Array([-0.425, 0.4]) });
    };
  }
  /**
   * Builds a label for flight plan waypoints.
   * @param color The color of the label.
   * @param displaySetting The 'mapWaypointsDisplay' setting.
   * @returns A new factory that will create the label.
   */
  static buildFlightPlanLabel(color, displaySetting) {
    return (w) => new FlightPathWaypointLabel(w, displaySetting, { fontSize: 24, fontColor: color, font: "WT21" });
  }
  /**
   * Builds an icon for a waypoint.
   * @param id The ID of the icon.
   * @param priority he render priority of this icon.
   * @returns A factory that builds the image icon.
   */
  static buildIcon(id, priority = WT21MapWaypointIconPriority.Bottom) {
    return (w) => new MapWaypointImageIcon(w, priority, ImageCache.get(id), MapSystemConfig.ICON_SIZE);
  }
  /**
   * Configures the map waypoint display layer.
   * @returns A builder function to configure the waypoint display system.
   *
   */
  static configureMapWaypoints() {
    return (builder) => {
      builder.withSearchCenter("target");
      MapSystemConfig.configWptRoles(MapSystemWaypointRoles.Normal, builder);
    };
  }
  /**
   * Configures the map waypoint role styles.
   * @param role The role to configure.
   * @param builder The waypoint display builder
   */
  static configWptRoles(role, builder) {
    builder
      .addDefaultIcon(role, MapSystemConfig.buildIcon("INTERSECTION"))
      .addDefaultLabel(role, MapSystemConfig.buildFacilityLabel(cyan))
      .addIcon(role, WaypointTypes.Airport, MapSystemConfig.buildIcon("AIRPORT"))
      .addIcon(role, WaypointTypes.NDB, MapSystemConfig.buildIcon("NDB"))
      .addIcon(role, WaypointTypes.VOR, (w) => {
        switch (w.facility.get().type) {
          case VorType.VOR:
            return new MapWaypointImageIcon(w, 0, ImageCache.get("VOR"), MapSystemConfig.ICON_SIZE);
          case VorType.VORDME:
            return new MapWaypointImageIcon(w, 0, ImageCache.get("VORDME"), MapSystemConfig.ICON_SIZE);
          case VorType.DME:
            return new MapWaypointImageIcon(w, 0, ImageCache.get("DME"), MapSystemConfig.ICON_SIZE);
          case VorType.TACAN:
            return new MapWaypointImageIcon(w, 0, ImageCache.get("TACAN"), MapSystemConfig.ICON_SIZE);
          default:
            return new MapWaypointImageIcon(w, 0, ImageCache.get("VORTAC"), MapSystemConfig.ICON_SIZE);
        }
      });
  }
  /**
   * Configures the map flight plan display layer.
   * @param bus The event bus to use.
   * @param waypointAlerter A waypoint alerter that will control the flash of the alering waypoint.
   * @param pfdOrMfd Whether this map is on the PFD or MFD.
   * @returns A builder function to configure the flight plan display system.
   */
  static configureFlightPlan(bus, waypointAlerter, pfdOrMfd) {
    return (builder) => {
      const effectiveLegIndex = Subject.create(-1);
      const sub = bus.getSubscriber();
      const settings = MapUserSettings.getAliasedManager(bus, pfdOrMfd);
      const showMissedAppr = () => BitFlags.isAll(settings.getSetting("mapWaypointsDisplay").value, MapWaypointsDisplay.MissedApproach);
      const isMissedApproachActive = Subject.create(false);
      sub.on("lnavdata_cdi_scale_label").handle((x) => isMissedApproachActive.set(x === CDIScaleLabel.MissedApproach));
      let currentActiveWaypointIcon;
      let currentActiveWaypointLabel;
      waypointAlerter.isDisplayed.sub((isDisplayed) => {
        currentActiveWaypointIcon === null || currentActiveWaypointIcon === void 0 ? void 0 : currentActiveWaypointIcon.setDisplayed(isDisplayed);
        currentActiveWaypointLabel === null || currentActiveWaypointLabel === void 0 ? void 0 : currentActiveWaypointLabel.setDisplayed(isDisplayed);
      });
      sub.on("lnavdata_nominal_leg_index").handle(effectiveLegIndex.set.bind(effectiveLegIndex));
      builder
        .registerRole(PlanWaypointRoles.Active)
        .registerRole(PlanWaypointRoles.Ahead)
        .registerRole(PlanWaypointRoles.From)
        .addDefaultIcon(PlanWaypointRoles.Ahead, MapSystemConfig.buildIcon("FLIGHTPLAN", WT21MapWaypointIconPriority.FlightPlan))
        .addDefaultIcon(PlanWaypointRoles.Active, (w) => {
          currentActiveWaypointIcon = new ActiveWaypointIcon(w, 999, ImageCache.get("FLIGHTPLAN_M"), MapSystemConfig.ICON_SIZE);
          return currentActiveWaypointIcon;
        })
        .addDefaultIcon(PlanWaypointRoles.From, MapSystemConfig.buildIcon("FLIGHTPLAN_C", WT21MapWaypointIconPriority.FlightPlan))
        .addDefaultLabel(PlanWaypointRoles.Ahead, MapSystemConfig.buildFlightPlanLabel(white, settings.getSetting("mapWaypointsDisplay")))
        .addDefaultLabel(PlanWaypointRoles.Active, (w) => {
          currentActiveWaypointLabel = new FlightPathWaypointLabel(w, settings.getSetting("mapWaypointsDisplay"), { fontSize: 24, fontColor: magenta, font: "WT21" });
          return currentActiveWaypointLabel;
        })
        .addDefaultLabel(PlanWaypointRoles.From, MapSystemConfig.buildFlightPlanLabel(cyan))
        .addLabel(PlanWaypointRoles.Ahead, WaypointTypes.FlightPlan, MapSystemConfig.buildFlightPlanLabel(white, settings.getSetting("mapWaypointsDisplay")))
        .addLabel(PlanWaypointRoles.Active, WaypointTypes.FlightPlan, (w) => {
          currentActiveWaypointLabel = new FlightPathWaypointLabel(w, settings.getSetting("mapWaypointsDisplay"), { fontSize: 24, fontColor: magenta, font: "WT21" });
          return currentActiveWaypointLabel;
        })
        .addLabel(PlanWaypointRoles.From, WaypointTypes.FlightPlan, MapSystemConfig.buildFlightPlanLabel(cyan));
      builder.withLegPathStyles((plan, leg, activeLeg, legIndex) => {
        const isMissedApproachLeg = BitFlags.isAll(leg.flags, LegDefinitionFlags.MissedApproach);
        const isHoldLeg = leg.leg.type === LegType.HF || leg.leg.type === LegType.HA;
        if (isMissedApproachLeg) {
          // We only want the MAP legs to be in cyan if we are not already in the missed approach
          if (!isMissedApproachActive.get()) {
            if (showMissedAppr()) {
              return isHoldLeg ? MapSystemConfig.HoldLegMapPath : MapSystemConfig.CyanPath;
            } else {
              return FlightPathRenderStyle.Hidden;
            }
          }
        }
        if (legIndex > effectiveLegIndex.get()) {
          return isHoldLeg ? MapSystemConfig.HoldLegPath : MapSystemConfig.WhitePath;
        } else if (legIndex === effectiveLegIndex.get()) {
          return isHoldLeg ? MapSystemConfig.HoldLegActivePath : MapSystemConfig.MagentaPath;
        } else if (legIndex === effectiveLegIndex.get() - 1) {
          return isHoldLeg ? MapSystemConfig.HoldLegPath : MapSystemConfig.WhitePath;
        }
        return FlightPathRenderStyle.Hidden;
      });
      builder.withLegWaypointRoles((plan, leg, activeLeg, legIndex) => {
        const isMissedApproachLeg = BitFlags.isAll(leg.flags, LegDefinitionFlags.MissedApproach);
        if (isMissedApproachLeg) {
          // We only want the MAP legs to be in cyan if we are not already in the missed approach
          if (!isMissedApproachActive.get()) {
            if (showMissedAppr()) {
              return builder.getRoleId(PlanWaypointRoles.From);
            } else {
              return 0;
            }
          }
        }
        if (legIndex > effectiveLegIndex.get()) {
          return builder.getRoleId(PlanWaypointRoles.Ahead);
        } else if (legIndex === effectiveLegIndex.get()) {
          return builder.getRoleId(PlanWaypointRoles.Active);
        } else if (legIndex === effectiveLegIndex.get() - 1) {
          return builder.getRoleId(PlanWaypointRoles.From);
        }
        return 0;
      });
    };
  }
  /**
   * Configures the map flight plan display layer for the mod flight plan.
   * @param bus The event bus to use.
   * @param pfdOrMfd Whether this map is on the PFD or MFD.
   * @returns A builder function to configure the mod flight plan display system.
   */
  static configureModFlightPlan(bus, pfdOrMfd) {
    return (builder) => {
      const settings = MapUserSettings.getAliasedManager(bus, pfdOrMfd);
      const showMissedAppr = () => BitFlags.isAll(settings.getSetting("mapWaypointsDisplay").value, MapWaypointsDisplay.MissedApproach);
      const isMissedApproachActive = Subject.create(false);
      bus
        .getSubscriber()
        .on("lnavdata_cdi_scale_label")
        .handle((x) => isMissedApproachActive.set(x === CDIScaleLabel.MissedApproach));
      const currentlyInMod = Subject.create(false);
      bus
        .getSubscriber()
        .on("fmcExecActive")
        .handle((active) => currentlyInMod.set(active === 1));
      builder
        .registerRole(PlanWaypointRoles.Mod)
        .addDefaultIcon(PlanWaypointRoles.Mod, MapSystemConfig.buildIcon("FLIGHTPLAN"))
        .addDefaultLabel(PlanWaypointRoles.Mod, MapSystemConfig.buildFacilityLabel(white))
        .addLabel(PlanWaypointRoles.Mod, WaypointTypes.FlightPlan, MapSystemConfig.buildFlightPlanLabel(white));
      builder.withLegPathStyles((plan, leg, activeLeg, legIndex, activeLegIndex) => {
        if (legIndex >= activeLegIndex && currentlyInMod.get()) {
          const isMissedApproachLeg = BitFlags.isAll(leg.flags, LegDefinitionFlags.MissedApproach);
          if (isMissedApproachLeg && !showMissedAppr() && !isMissedApproachActive.get()) {
            return FlightPathRenderStyle.Hidden;
          }
          const isHoldLeg = leg.leg.type === LegType.HF || leg.leg.type === LegType.HA;
          return isHoldLeg ? MapSystemConfig.HoldLegDashedPath : MapSystemConfig.WhiteDashedPath;
        }
        return FlightPathRenderStyle.Hidden;
      });
      builder.withLegWaypointRoles((plan, leg, activeLeg, legIndex, activeLegIndex) => {
        if (legIndex >= activeLegIndex && currentlyInMod.get()) {
          const isMissedApproachLeg = BitFlags.isAll(leg.flags, LegDefinitionFlags.MissedApproach);
          if (isMissedApproachLeg && !showMissedAppr() && !isMissedApproachActive.get()) {
            return 0;
          }
          return builder.getRoleId(PlanWaypointRoles.Mod);
        }
        return 0;
      });
    };
  }
  /**
   * Creates an icon for a traffic intruder.
   * @param intruder The intruder for which to create an icon.
   * @param context The context of the icon's parent map.
   * @returns An icon for the specified intruder.
   */
  static createTrafficIntruderIcon(intruder, context) {
    return new MapTrafficIntruderIcon(intruder, context.model.getModule(MapSystemKeys.Traffic), context.model.getModule(MapSystemKeys.OwnAirplaneProps));
  }
  /**
   * Initializes global canvas styles for the traffic layer.
   * @param context The canvas rendering context for which to initialize styles.
   */
  static initTrafficLayerCanvasStyles(context) {
    context.textAlign = "center";
    context.font = "Arial";
  }
}
MapSystemConfig.ICON_SIZE = Vec2Math.create(40, 40);
MapSystemConfig.MagentaPath = {
  isDisplayed: true,
  width: 4,
  style: magenta,
};
MapSystemConfig.WhitePath = {
  isDisplayed: true,
  width: 4,
  style: white,
};
MapSystemConfig.WhiteDashedPath = {
  isDisplayed: true,
  width: 4,
  style: white,
  dash: [14, 10],
};
MapSystemConfig.CyanPath = {
  isDisplayed: true,
  width: 4,
  style: cyan,
};
MapSystemConfig.HoldLegPath = {
  partsToRender: FlightPathLegRenderPart.Base | FlightPathLegRenderPart.Ingress,
  styleBuilder: MapSystemConfig.buildWhiteHoldStyle,
};
MapSystemConfig.HoldLegActivePath = {
  partsToRender: FlightPathLegRenderPart.Base | FlightPathLegRenderPart.Ingress,
  styleBuilder: MapSystemConfig.buildMagentaHoldStyle,
};
MapSystemConfig.HoldLegMapPath = {
  partsToRender: FlightPathLegRenderPart.Base | FlightPathLegRenderPart.Ingress,
  styleBuilder: MapSystemConfig.buildCyanHoldStyle,
};
MapSystemConfig.HoldLegDashedPath = {
  partsToRender: FlightPathLegRenderPart.Base | FlightPathLegRenderPart.Ingress,
  styleBuilder: MapSystemConfig.buildWhiteDashedHoldStyle,
};
```

</details>

Add the ImageCache class to be able to retrieve the right image based on id

<details>

<summary>class ImageCache</summary>
  class ImageCache {
  /**
   * Loads the image from the url and adds it to the cache.
   * @static
   * @param key The image key to access it later.
   * @param url The url to load the image from.
   */
  static addToCache(key, url) {
    if (this.cache[key] === undefined) {
      const img = new Image();
      img.src = url;
      this.cache[key] = img;
    }
  }
  /**
   * Gets a cached image element.
   * @static
   * @param key The key of the cached image.
   * @returns The cached image element.
   */
  static get(key) {
    return this.cache[key];
  }
}
ImageCache.cache = {};
//coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/plane-circles.svg"
ImageCache.addToCache("AIRPORT", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/airport_c.png");
ImageCache.addToCache("INTERSECTION", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/intersection.png");
ImageCache.addToCache("NDB", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/ndb.png");
ImageCache.addToCache("VOR", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/vor.png");
ImageCache.addToCache("DME", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/dme.png");
ImageCache.addToCache("VORDME", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/vordme.png");
ImageCache.addToCache("VORTAC", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/vortac.png");
ImageCache.addToCache("TACAN", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/tacan.png");
ImageCache.addToCache("FLIGHTPLAN", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/flightplan.png");
ImageCache.addToCache("FLIGHTPLAN_M", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/flightplan_m.png");
ImageCache.addToCache("FLIGHTPLAN_C", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/flightplan_c.png");
ImageCache.addToCache("TOD", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/tod.png");
</details>

Usage:
```js
//add image:
ImageCache.addToCache("AIRPORT", "coui://html_ui/Pages/VCockpit/Instruments/Miltech_U2S/Assets/icons/airport_c.png");
//retrieve the image:
ImageCache.get(id)
```

The images used in the u2 project are included in the repo
