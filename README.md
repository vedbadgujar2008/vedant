package com.sololeveling.app

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.animation.*
import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Settings
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.scale
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import kotlinx.coroutines.delay

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            SoloLevelingTheme {
                SoloLevelingApp()
            }
        }
    }
}

@Composable
fun SoloLevelingApp() {
    var currentScreen by remember { mutableStateOf("dashboard") }
    var playerData by remember {
        mutableStateOf(PlayerDataEnhanced(
            name = "Sung Jin-woo",
            rank = "E-Rank",
            level = 5,
            hp = 85,
            maxHp = 100,
            xp = 420,
            maxXp = 1000,
            fatigue = 35,
            maxFatigue = 100,
            strength = 12,
            agility = 14,
            sense = 10,
            vitality = 16,
            intelligence = 11,
            gold = 250,
            systemSynchronization = 100,
            dailyStreak = 3,
            totalWorkoutTime = 42,
            lastWorkoutType = "Strength Training"
        ))
    }

    Surface(
        modifier = Modifier.fillMaxSize(),
        color = Color(0xFF0a0e27)
    ) {
        Box(modifier = Modifier.fillMaxSize()) {
            when (currentScreen) {
                "dashboard" -> EnhancedDashboard(playerData) { newScreen, updatedData ->
                    currentScreen = newScreen
                    if (updatedData != null) playerData = updatedData
                }
                "quests" -> DynamicQuestsScreen(playerData) { newScreen, updatedData ->
                    currentScreen = newScreen
                    if (updatedData != null) playerData = updatedData
                }
                "inventory" -> InventoryScreenEnhanced(playerData) { currentScreen = it }
                "shop" -> ShopScreenEnhanced(playerData) { currentScreen = it }
                "rank-up" -> RankUpExamination(playerData) { newScreen, updatedData ->
                    currentScreen = newScreen
                    if (updatedData != null) playerData = updatedData
                }
            }
        }
    }
}

// ========================
// ENHANCED DASHBOARD
// ========================
@Composable
fun EnhancedDashboard(
    playerData: PlayerDataEnhanced,
    onNavigate: (String, PlayerDataEnhanced?) -> Unit
) {
    var questProgress by remember {
        mutableStateOf(listOf(
            DynamicQuestItem("${playerData.currentDailyQuests.pushups} Push-ups", false, "Strength", 1),
            DynamicQuestItem("${playerData.currentDailyQuests.situps} Sit-ups", false, "Core", 2),
            DynamicQuestItem("${playerData.currentDailyQuests.squats} Squats", false, "Legs", 3),
            DynamicQuestItem("${String.format("%.1f", playerData.currentDailyQuests.distance)} km Run", false, "Cardio", 4)
        ))
    }

    var gateOpened by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .verticalScroll(rememberScrollState())
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        // CINEMATIC HEADER
        CinematicHeader(playerData)

        // SYSTEM STATUS
        SystemStatusBar(playerData.systemSynchronization)

        // STAT DISPLAY WITH HUD STYLE
        HUDStatsSection(playerData)

        // ATTRIBUTES GRID
        AttributesGridEnhanced(playerData)

        // DYNAMIC DAILY QUEST
        DynamicDailyQuestBox(playerData, questProgress) { updatedProgress, updatedData ->
            questProgress = updatedProgress
            onNavigate("dashboard", updatedData)
        }

        // STREAK & HISTORY
        StreakAndHistorySection(playerData)

        // ACTION BUTTONS WITH CINEMATIC FEEL
        AnimatedGateButton(gateOpened) {
            gateOpened = true
            onNavigate("quests", null)
        }

        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Button(
                onClick = { onNavigate("inventory", null) },
                modifier = Modifier
                    .weight(1f)
                    .height(48.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color(0xFF4a4a6a),
                    contentColor = Color(0xFF00d4ff)
                ),
                shape = RoundedCornerShape(6.dp)
            ) {
                Text("🎒 Inventory", fontWeight = FontWeight.Bold)
            }

            Button(
                onClick = { onNavigate("shop", null) },
                modifier = Modifier
                    .weight(1f)
                    .height(48.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color(0xFF1a6b5a),
                    contentColor = Color(0xFF00d4ff)
                ),
                shape = RoundedCornerShape(6.dp)
            ) {
                Text("🛍️ Shop", fontWeight = FontWeight.Bold)
            }
        }

        // RANK UP BUTTON
        if (playerData.level % 5 == 0) {
            Button(
                onClick = { onNavigate("rank-up", null) },
                modifier = Modifier
                    .fillMaxWidth()
                    .height(56.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color(0xFFff6b6b),
                    contentColor = Color(0xFF0a0e27)
                ),
                shape = RoundedCornerShape(8.dp)
            ) {
                Text("⚠️ RANK-UP EXAMINATION AVAILABLE", fontWeight = FontWeight.Bold, fontSize = 14.sp)
            }
        }

        Spacer(modifier = Modifier.height(80.dp))
    }

    BottomNavigationEnhanced(currentScreen = "dashboard", onNavigate = onNavigate)
}

@Composable
fun CinematicHeader(playerData: PlayerDataEnhanced) {
    var pulseScale by remember { mutableStateOf(1f) }

    LaunchedEffect(Unit) {
        while (true) {
            delay(800)
            pulseScale = 1.05f
            delay(400)
            pulseScale = 1f
        }
    }

    Column(
        modifier = Modifier
            .fillMaxWidth()
            .border(2.dp, Color(0xFF00d4ff))
            .background(Color(0xFF00d4ff).copy(alpha = 0.05f))
            .padding(16.dp)
            .scale(pulseScale),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "⟨ SYSTEM ACTIVATED ⟩",
            fontSize = 10.sp,
            color = Color(0xFF4ade80),
            fontFamily = FontFamily.Monospace,
            fontWeight = FontWeight.Bold,
            letterSpacing = 1.sp
        )

        Text(
            text = playerData.name,
            fontSize = 28.sp,
            fontWeight = FontWeight.Bold,
            color = Color(0xFF00d4ff),
            letterSpacing = 2.sp
        )

        Text(
            text = "${playerData.rank} HUNTER | LVL ${playerData.level}",
            fontSize = 12.sp,
            color = Color(0xFF00a8cc),
            fontWeight = FontWeight.Bold,
            fontFamily = FontFamily.Monospace
        )

        Spacer(modifier = Modifier.height(8.dp))

        Text(
            text = "Last Activity: ${playerData.lastWorkoutType}",
            fontSize = 10.sp,
            color = Color(0xFF00d4ff).copy(alpha = 0.7f),
            fontFamily = FontFamily.Monospace
        )
    }
}

@Composable
fun SystemStatusBar(syncPercentage: Int) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .border(1.dp, Color(0xFF00d4ff))
            .background(Color(0xFF1a1f3a))
            .padding(12.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = "SYSTEM SYNC:",
            fontSize = 11.sp,
            color = Color(0xFF00a8cc),
            fontFamily = FontFamily.Monospace,
            fontWeight = FontWeight.Bold
        )

        Row(
            modifier = Modifier.weight(1f),
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            LinearProgressIndicator(
                progress = syncPercentage / 100f,
                modifier = Modifier
                    .weight(1f)
                    .height(6.dp),
                color = when {
                    syncPercentage >= 80 -> Color(0xFF4ade80)
                    syncPercentage >= 50 -> Color(0xFFffa500)
                    else -> Color(0xFFff6b6b)
                },
                trackColor = Color(0xFF00d4ff).copy(alpha = 0.2f)
            )

            Text(
                text = "$syncPercentage%",
                fontSize = 11.sp,
                color = when {
                    syncPercentage >= 80 -> Color(0xFF4ade80)
                    syncPercentage >= 50 -> Color(0xFFffa500)
                    else -> Color(0xFFff6b6b)
                },
                fontFamily = FontFamily.Monospace,
                fontWeight = FontWeight.Bold
            )
        }
    }
}

@Composable
fun HUDStatsSection(playerData: PlayerDataEnhanced) {
    Column(
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        HUDStatRow("HP", playerData.hp, playerData.maxHp, Color(0xFFff6b6b))
        HUDStatRow("XP", playerData.xp, playerData.maxXp, Color(0xFF00d4ff))
        HUDStatRow("FATIGUE", playerData.fatigue, playerData.maxFatigue, Color(0xFFffa500))
    }
}

@Composable
fun HUDStatRow(label: String, current: Int, max: Int, barColor: Color) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .border(1.dp, Color(0xFF00d4ff))
            .background(Color(0xFF00d4ff).copy(alpha = 0.02f))
            .padding(10.dp)
    ) {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text(
                text = "⟨ $label ⟩",
                fontSize = 11.sp,
                fontWeight = FontWeight.Bold,
                color = barColor,
                fontFamily = FontFamily.Monospace
            )
            Text(
                text = "$current / $max",
                fontSize = 11.sp,
                fontWeight = FontWeight.Bold,
                color = barColor,
                fontFamily = FontFamily.Monospace
            )
        }

        Spacer(modifier = Modifier.height(6.dp))

        LinearProgressIndicator(
            progress = current.toFloat() / max.toFloat(),
            modifier = Modifier
                .fillMaxWidth()
                .height(6.dp),
            color = barColor,
            trackColor = Color(0xFF00d4ff).copy(alpha = 0.15f)
        )
    }
}

@Composable
fun AttributesGridEnhanced(playerData: PlayerDataEnhanced) {
    Column(
        modifier = Modifier.fillMaxWidth(),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            AttributeCardEnhanced("STR", playerData.strength, Modifier.weight(1f))
            AttributeCardEnhanced("AGI", playerData.agility, Modifier.weight(1f))
        }

        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            AttributeCardEnhanced("SNS", playerData.sense, Modifier.weight(1f))
            AttributeCardEnhanced("VIT", playerData.vitality, Modifier.weight(1f))
        }

        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            AttributeCardEnhanced("INT", playerData.intelligence, Modifier.weight(1f))
            AttributeCardEnhanced("GLD", playerData.gold, Modifier.weight(1f))
        }
    }
}

@Composable
fun AttributeCardEnhanced(name: String, value: Int, modifier: Modifier = Modifier) {
    Column(
        modifier = modifier
            .border(1.dp, Color(0xFF00d4ff))
            .background(Color(0xFF00d4ff).copy(alpha = 0.03f))
            .padding(12.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = name,
            fontSize = 10.sp,
            color = Color(0xFF00a8cc),
            fontWeight = FontWeight.Bold,
            fontFamily = FontFamily.Monospace
        )
        Spacer(modifier = Modifier.height(4.dp))
        Text(
            text = value.toString(),
            fontSize = 18.sp,
            color = Color(0xFF00d4ff),
            fontWeight = FontWeight.Bold,
            fontFamily = FontFamily.Monospace
        )
    }
}

@Composable
fun DynamicDailyQuestBox(
    playerData: PlayerDataEnhanced,
    questProgress: List<DynamicQuestItem>,
    onQuestUpdate: (List<DynamicQuestItem>, PlayerDataEnhanced) -> Unit
) {
    val completedCount = questProgress.count { it.completed }
    val difficulty = playerData.getCurrentQuestDifficulty()

    Column(
        modifier = Modifier
            .fillMaxWidth()
            .border(2.dp, Color(0xFFff6b6b))
            .background(Color(0xFFff6b6b).copy(alpha = 0.08f))
            .padding(14.dp),
        verticalArrangement = Arrangement.spacedBy(10.dp)
    ) {
        // Quest Header
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(
                text = "⟨ DAILY QUEST ⟩",
                fontSize = 12.sp,
                color = Color(0xFFff6b6b),
                fontWeight = FontWeight.Bold,
                fontFamily = FontFamily.Monospace
            )

            Text(
                text = difficulty,
                fontSize = 10.sp,
                color = when (difficulty) {
                    "EASY" -> Color(0xFF4ade80)
                    "MEDIUM" -> Color(0xFFffa500)
                    "HARD" -> Color(0xFFff6b6b)
                    "VERY HARD" -> Color(0xFF9d4edd)
                    else -> Color(0xFF00d4ff)
                },
                fontWeight = FontWeight.Bold,
                fontFamily = FontFamily.Monospace
            )
        }

        // Quest Items
        questProgress.forEach { quest ->
            Row(
                modifier = Modifier.fillMaxWidth(),
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                Checkbox(
                    checked = quest.completed,
                    onCheckedChange = { isChecked ->
                        val updatedProgress = questProgress.map {
                            if (it.id == quest.id) it.copy(completed = isChecked) else it
                        }

                        val xpGain = if (isChecked) {
                            when (difficulty) {
                                "EASY" -> 25
                                "MEDIUM" -> 50
                                "HARD" -> 100
                                "VERY HARD" -> 150
                                else -> 50
                            }
                        } else 0

                        var updatedData = playerData.copy(
                            xp = (playerData.xp + xpGain).coerceAtMost(playerData.maxXp)
                        )

                        if (updatedProgress.all { it.completed }) {
                            updatedData = updatedData.copy(
                                systemSynchronization = (updatedData.systemSynchronization + 5).coerceAtMost(100),
                                dailyStreak = updatedData.dailyStreak + 1
                            )
                        }

                        onQuestUpdate(updatedProgress, updatedData)
                    },
                    modifier = Modifier.size(18.dp),
                    colors = CheckboxDefaults.colors(
                        checkedColor = Color(0xFFff6b6b),
                        uncheckedColor = Color(0xFFff6b6b)
                    )
                )

                Text(
                    text = quest.name,
                    fontSize = 12.sp,
                    color = Color(0xFF00d4ff)
                )

                Spacer(modifier = Modifier.weight(1f))

                Text(
                    text = "[${quest.type}]",
                    fontSize = 9.sp,
                    color = Color(0xFF00a8cc),
                    fontFamily = FontFamily.Monospace
                )
            }
        }

        // Progress Bar
        Spacer(modifier = Modifier.height(8.dp))
        LinearProgressIndicator(
            progress = completedCount.toFloat() / questProgress.size,
            modifier = Modifier
                .fillMaxWidth()
                .height(5.dp),
            color = Color(0xFFff6b6b),
            trackColor = Color(0xFFff6b6b).copy(alpha = 0.2f)
        )

        Text(
            text = "PROGRESS: $completedCount/${questProgress.size}",
            fontSize = 10.sp,
            color = Color(0xFF00a8cc),
            fontWeight = FontWeight.Bold,
            fontFamily = FontFamily.Monospace
        )
    }
}

@Composable
fun StreakAndHistorySection(playerData: PlayerDataEnhanced) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .border(1.dp, Color(0xFF00d4ff))
            .background(Color(0xFF00d4ff).copy(alpha = 0.03f))
            .padding(12.dp),
        horizontalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Column(modifier = Modifier.weight(1f)) {
            Text(
                text = "STREAK",
                fontSize = 10.sp,
                color = Color(0xFF00a8cc),
                fontFamily = FontFamily.Monospace,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = "${playerData.dailyStreak} Days",
                fontSize = 18.sp,
                color = Color(0xFF4ade80),
                fontWeight = FontWeight.Bold
            )
        }

        Column(modifier = Modifier.weight(1f)) {
            Text(
                text = "TOTAL TIME",
                fontSize = 10.sp,
                color = Color(0xFF00a8cc),
                fontFamily = FontFamily.Monospace,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = "${playerData.totalWorkoutTime}h",
                fontSize = 18.sp,
                color = Color(0xFF4ade80),
                fontWeight = FontWeight.Bold
            )
        }

        Column(modifier = Modifier.weight(1f)) {
            Text(
                text = "LAST ACTIVITY",
                fontSize = 10.sp,
                color = Color(0xFF00a8cc),
                fontFamily = FontFamily.Monospace,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = playerData.lastWorkoutType.take(8),
                fontSize = 11.sp,
                color = Color(0xFF00d4ff),
                fontWeight = FontWeight.Bold
            )
        }
    }
}

@Composable
fun AnimatedGateButton(isOpened: Boolean, onClick: () -> Unit) {
    var scale by remember { mutableStateOf(1f) }

    LaunchedEffect(isOpened) {
        if (!isOpened) {
            scale = 1f
        }
    }

    Button(
        onClick = { if (!isOpened) onClick() },
        modifier = Modifier
            .fillMaxWidth()
            .height(56.dp)
            .scale(scale),
        colors = ButtonDefaults.buttonColors(
            containerColor = Color(0xFF00d4ff),
            contentColor = Color(0xFF0a0e27)
        ),
        shape = RoundedCornerShape(8.dp)
    ) {
        Text("⟨ OPEN GATE ⟩ DUNGEON", fontWeight = FontWeight.Bold, fontSize = 14.sp)
    }
}

// ========================
// DYNAMIC QUESTS SCREEN
// ========================
@Composable
fun DynamicQuestsScreen(
    playerData: PlayerDataEnhanced,
    onNavigate: (String, PlayerDataEnhanced?) -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(Color(0xFF0a0e27))
            .padding(16.dp)
            .verticalScroll(rememberScrollState())
    ) {
        Text(
            text = "⟨ AVAILABLE GATES ⟩",
            fontSize = 20.sp,
            fontWeight = FontWeight.Bold,
            color = Color(0xFF00d4ff),
            fontFamily = FontFamily.Monospace,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        DungeonCardEnhanced(
            title = "INSTANT DUNGEON",
            description = "25-min focus session. Defeat the goblin boss!",
            reward = "150 XP + 50 Gold",
            icon = "🪨",
            difficulty = "E-RANK"
        )

        DungeonCardEnhanced(
            title = "RAID GATE",
            description = "Extreme fitness challenge. 10 km run + 100 push-ups.",
            reward = "500 XP + 200 Gold + Title",
            icon = "🐉",
            difficulty = "S-RANK"
        )

        DungeonCardEnhanced(
            title = "EVENT QUEST",
            description = "Study marathon - 2 hours straight.",
            reward = "300 XP + Rare Item",
            icon = "📖",
            difficulty = "B-RANK"
        )

        Spacer(modifier = Modifier.weight(1f))

        Button(
            onClick = { onNavigate("dashboard", null) },
            modifier = Modifier
                .fillMaxWidth()
                .height(48.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = Color(0xFF00d4ff),
                contentColor = Color(0xFF0a0e27)
            ),
            shape = RoundedCornerShape(6.dp)
        ) {
            Text("← CLOSE GATE", fontWeight = FontWeight.Bold)
        }

        Spacer(modifier = Modifier.height(80.dp))
    }

    BottomNavigationEnhanced(currentScreen = "quests", onNavigate = onNavigate)
}

@Composable
fun DungeonCardEnhanced(
    title: String,
    description: String,
    reward: String,
    icon: String,
    difficulty: String
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(bottom = 12.dp),
        colors = CardDefaults.cardColors(containerColor = Color(0xFF1a1f3a)),
        border = BorderStroke(1.dp, Color(0xFF00d4ff))
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(
                    text = "$icon $title",
                    fontSize = 14.sp,
                    fontWeight = FontWeight.Bold,
                    color = Color(0xFF00d4ff),
                    fontFamily = FontFamily.Monospace
                )
                Text(
                    text = difficulty,
                    fontSize = 10.sp,
                    color = Color(0xFFff6b6b),
                    fontWeight = FontWeight.Bold,
                    fontFamily = FontFamily.Monospace
                )
            }

            Text(
                text = description,
                fontSize = 12.sp,
                color = Color(0xFF00a8cc)
            )

            Text(
                text = "⟨ REWARD: $reward ⟩",
                fontSize = 11.sp,
                color = Color(0xFF4ade80),
                fontWeight = FontWeight.Bold,
                fontFamily = FontFamily.Monospace
            )
        }
    }
}

// ========================
// RANK-UP EXAMINATION
// ========================
@Composable
fun RankUpExamination(
    playerData: PlayerDataEnhanced,
    onNavigate: (String, PlayerDataEnhanced?) -> Unit
) {
    var examinationStarted by remember { mutableStateOf(false) }
    var passed by remember { mutableStateOf(false) }
    var syncDecreased by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(Color(0xFF0a0e27))
            .padding(16.dp)
            .verticalScroll(rememberScrollState()),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "⟨ RANK-UP EXAMINATION ⟩",
            fontSize = 22.sp,
            fontWeight = FontWeight.Bold,
            color = Color(0xFFffa500),
            fontFamily = FontFamily.Monospace
        )

        if (!examinationStarted) {
            Text(
                text = "You are eligible for the next rank.",
                fontSize = 14.sp,
                color = Color(0xFF00d4ff)
            )

            Text(
                text = "Complete a challenging quest to prove your strength.",
                fontSize = 12.sp,
                color = Color(0xFF00a8cc)
            )

            Button(
                onClick = { examinationStarted = true },
                modifier = Modifier
                    .fillMaxWidth()
                    .height(56.dp),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color(0xFFffa500),
                    contentColor = Color(0xFF0a0e27)
                ),
                shape = RoundedCornerShape(8.dp)
            ) {
                Text("BEGIN EXAMINATION", fontWeight = FontWeight.Bold, fontSize = 14.sp)
            }
        } else {
            Text(
                text = "Complete 30 minutes of intense workout to pass!",
                fontSize = 13.sp,
                color = Color(0xFF4ade80)
            )

            AnimatedCheckmark(passed)

            if (passed) {
                Text(
                    text = "✓ EXAMINATION PASSED!",
                    fontSize = 16.sp,
                    color = Color(0xFF4ade80),
                    fontWeight = FontWeight.Bold
                )

                Button(
                    onClick = {
                        val updatedData = playerData.copy(
                            level = playerData.level + 1,
                            xp = 0,
                            maxXp = (playerData.maxXp * 1.2).toInt()
                        )
                        onNavigate("dashboard", updatedData)
                    },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(48.dp),
                    colors = ButtonDefaults.buttonColors(
                        containerColor = Color(0xFF4ade80),
                        contentColor = Color(0xFF0a0e27)
                    ),
                    shape = RoundedCornerShape(6.dp)
                ) {
                    Text("CLAIM REWARD", fontWeight = FontWeight.Bold)
                }
            } else {
                Button(
                    onClick = { passed = true },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(48.dp),
                    colors = ButtonDefaults.buttonColors(
                        containerColor = Color(0xFFff6b6b),
                        contentColor = Color(0xFF0a0e27)
                    ),
                    shape = RoundedCornerShape(6.dp)
                ) {
                    Text("✓ COMPLETED WORKOUT", fontWeight = FontWeight.Bold)
                }
            }
        }

        Spacer(modifier = Modifier.weight(1f))

        Button(
            onClick = { onNavigate("dashboard", null) },
            modifier = Modifier
                .fillMaxWidth()
                .height(48.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = Color(0xFF4a4a6a),
                contentColor = Color(0xFF00d4ff)
            ),
            shape = RoundedCornerShape(6.dp)
        ) {
            Text("← BACK", fontWeight = FontWeight.Bold)
        }

        Spacer(modifier = Modifier.height(80.dp))
    }

    BottomNavigationEnhanced(currentScreen = "rank-up", onNavigate = onNavigate)
}

@Composable
fun AnimatedCheckmark(isVisible: Boolean) {
    if (isVisible) {
        Text(
            text = "✓",
            fontSize = 64.sp,
            color = Color(0xFF4ade80),
            modifier = Modifier.scale(1.2f)
        )
    }
}

// ========================
// INVENTORY & SHOP SCREENS
// ========================
@Composable
fun InventoryScreenEnhanced(playerData: PlayerDataEnhanced, onNavigate: (String) -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(Color(0xFF0a0e27))
            .padding(16.dp)
            .verticalScroll(rememberScrollState())
    ) {
        Text(
            text = "⟨ INVENTORY ⟩",
            fontSize = 20.sp,
            fontWeight = FontWeight.Bold,
            color = Color(0xFF00d4ff),
            fontFamily = FontFamily.Monospace,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        InventoryItemEnhanced("Iron Dagger", "Common Weapon", "⚔️")
        InventoryItemEnhanced("Coffee Potion", "Consumable - Focus +20%", "☕")
        InventoryItemEnhanced("Dungeon Key", "Opens Instant Dungeon", "🔑")
        InventoryItemEnhanced("Health Potion", "Recovers 20 HP", "💊")
        InventoryItemEnhanced("Shadow", "Extracted Monster", "👥")

        Spacer(modifier = Modifier.weight(1f))

        Button(
            onClick = { onNavigate("dashboard") },
            modifier = Modifier
                .fillMaxWidth()
                .height(48.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = Color(0xFF00d4ff),
                contentColor = Color(0xFF0a0e27)
            ),
            shape = RoundedCornerShape(6.dp)
        ) {
            Text("← BACK", fontWeight = FontWeight.Bold)
        }

        Spacer(modifier = Modifier.height(80.dp))
    }

    BottomNavigationEnhanced(currentScreen = "inventory", onNavigate = onNavigate)
}

@Composable
fun InventoryItemEnhanced(name: String, description: String, emoji: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(bottom = 12.dp),
        colors = CardDefaults.cardColors(containerColor = Color(0xFF1a1f3a)),
        border = BorderStroke(1.dp, Color(0xFF00d4ff))
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(emoji, fontSize = 32.sp)
            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = name,
                    fontSize = 13.sp,
                    fontWeight = FontWeight.Bold,
                    color = Color(0xFF00d4ff),
                    fontFamily = FontFamily.Monospace
                )
                Text(
                    text = description,
                    fontSize = 11.sp,
                    color = Color(0xFF00a8cc)
                )
            }
        }
    }
}

@Composable
fun ShopScreenEnhanced(playerData: PlayerDataEnhanced, onNavigate: (String) -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(Color(0xFF0a0e27))
            .padding(16.dp)
            .verticalScroll(rememberScrollState())
    ) {
        Text(
            text = "⟨ SYSTEM SHOP ⟩",
            fontSize = 20.sp,
            fontWeight = FontWeight.Bold,
            color = Color(0xFF00d4ff),
            fontFamily = FontFamily.Monospace,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        ShopItemEnhanced("Dark Theme", "Profile theme", 100, "🎨")
        ShopItemEnhanced("Sword Skin", "Weapon cosmetic", 250, "⚔️")
        ShopItemEnhanced("Recovery Potion", "Restore 50 HP", 75, "💊")
        ShopItemEnhanced("Shadow Extract", "Skill unlock", 500, "👥")
        ShopItemEnhanced("Gold Chest", "500 bonus Gold", 150, "💰")

        Spacer(modifier = Modifier.weight(1f))

        Text(
            text = "💰 ${playerData.gold} GOLD",
            fontSize = 13.sp,
            color = Color(0xFF4ade80),
            fontWeight = FontWeight.Bold,
            fontFamily = FontFamily.Monospace,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        Button(
            onClick = { onNavigate("dashboard") },
            modifier = Modifier
                .fillMaxWidth()
                .height(48.dp),
            colors = ButtonDefaults.buttonColors(
                containerColor = Color(0xFF00d4ff),
                contentColor = Color(0xFF0a0e27)
            ),
            shape = RoundedCornerShape(6.dp)
        ) {
            Text("← BACK", fontWeight = FontWeight.Bold)
        }

        Spacer(modifier = Modifier.height(80.dp))
    }

    BottomNavigationEnhanced(currentScreen = "shop", onNavigate = onNavigate)
}

@Composable
fun ShopItemEnhanced(name: String, description: String, price: Int, emoji: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(bottom = 12.dp),
        colors = CardDefaults.cardColors(containerColor = Color(0xFF1a1f3a)),
        border = BorderStroke(1.dp, Color(0xFFffa500))
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.spacedBy(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(emoji, fontSize = 32.sp)
            Column(modifier = Modifier.weight(1f)) {
                Text(
                    text = name,
                    fontSize = 13.sp,
                    fontWeight = FontWeight.Bold,
                    color = Color(0xFF00d4ff),
                    fontFamily = FontFamily.Monospace
                )
                Text(
                    text = description,
                    fontSize = 11.sp,
                    color = Color(0xFF00a8cc)
                )
            }
            Text(
                text = "$price",
                fontSize = 12.sp,
                fontWeight = FontWeight.Bold,
                color = Color(0xFF4ade80),
                fontFamily = FontFamily.Monospace
            )
        }
    }
}

// ========================
// BOTTOM NAVIGATION
// ========================
@Composable
fun BoxScope.BottomNavigationEnhanced(currentScreen: String, onNavigate: (String, PlayerDataEnhanced?) -> Unit) {
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .align(Alignment.BottomCenter)
            .background(Color(0xFF0a0e27))
            .border(1.dp, Color(0xFF00d4ff).copy(alpha = 0.3f))
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(12.dp),
            horizontalArrangement = Arrangement.SpaceEvenly,
            verticalAlignment = Alignment.CenterVertically
        ) {
            NavButtonEnhanced("📊", "Dashboard", "dashboard" == currentScreen) {
                onNavigate("dashboard", null)
            }
            NavButtonEnhanced("⚡", "Quests", "quests" == currentScreen) {
                onNavigate("quests", null)
            }
            NavButtonEnhanced("🎒", "Inventory", "inventory" == currentScreen) {
                onNavigate("inventory", null)
            }
            NavButtonEnhanced("🛍️", "Shop", "shop" == currentScreen) {
                onNavigate("shop", null)
            }
        }
    }
}

@Composable
fun NavButtonEnhanced(emoji: String, label: String, isActive: Boolean, onClick: () -> Unit) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier.clickable(enabled = true) { onClick() }
    ) {
        Text(emoji, fontSize = 20.sp)
        Text(
            text = label,
            fontSize = 10.sp,
            color = if (isActive) Color(0xFF00d4ff) else Color(0xFF00a8cc),
            fontFamily = FontFamily.Monospace,
            fontWeight = if (isActive) FontWeight.Bold else FontWeight.Normal
        )
    }
}

// ========================
// THEME
// ========================
@Composable
fun SoloLevelingTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = darkColorScheme(
            primary = Color(0xFF00d4ff),
            secondary = Color(0xFF4a4a6a),
            background = Color(0xFF0a0e27)
        )
    ) {
        content()
    }
}

// ========================
// DATA MODELS
// ========================
data class PlayerDataEnhanced(
    val name: String,
    val rank: String,
    val level: Int,
    val hp: Int,
    val maxHp: Int,
    val xp: Int,
    val maxXp: Int,
    val fatigue: Int,
    val maxFatigue: Int,
    val strength: Int,
    val agility: Int,
    val sense: Int,
    val vitality: Int,
    val intelligence: Int,
    val gold: Int,
    val systemSynchronization: Int,
    val dailyStreak: Int,
    val totalWorkoutTime: Int,
    val lastWorkoutType: String,
    val currentDailyQuests: DailyQuestStats = getDailyQuestsForLevel(level)
) {
    fun getCurrentQuestDifficulty(): String {
        return when {
            level < 5 -> "EASY"
            level < 15 -> "MEDIUM"
            level < 30 -> "HARD"
            else -> "VERY HARD"
        }
    }
}

data class DynamicQuestItem(
    val name: String,
    val completed: Boolean,
    val type: String,
    val id: Int
)

data class DailyQuestStats(
    val pushups: Int = 10,
    val situps: Int = 15,
    val squats: Int = 15,
    val distance: Double = 1.5
)

fun getDailyQuestsForLevel(level: Int): DailyQuestStats {
    return when {
        level < 5 -> DailyQuestStats(10, 15, 15, 1.5)
        level < 10 -> DailyQuestStats(20, 30, 30, 3.0)
        level < 20 -> DailyQuestStats(50, 50, 50, 5.0)
        level < 30 -> DailyQuestStats(75, 75, 75, 7.0)
        else -> DailyQuestStats(100, 100, 100, 10.0)
    }
}
